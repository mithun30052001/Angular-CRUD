https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


panel-schedule-dialog.component.html

<div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="secondaryEmployeeIds" class="form-label"
              >Secondary panel members</label
            >
            <mat-form-field appearance="outline">
              <mat-chip-grid #chipGrid aria-label="Secondary panel selection">
                <mat-chip-row
                  *ngFor="let fruit of selectedPanel"
                  (removed)="remove(fruit.employeeId)"
                >
                  {{ fruit.employeeName }}
                  <button matChipRemove [attr.aria-label]="'remove ' + fruit">
                    <app-icon icon="close_small"></app-icon>
                  </button>
                </mat-chip-row>
              </mat-chip-grid>
              <input
                placeholder="Search secondary panel members"
                #secAutoCompleteInput
                [formControl]="secondaryPanelAcControl"
                [matChipInputFor]="chipGrid"
                [matAutocomplete]="secondaryPanelAuto"
              />
              <mat-autocomplete
                [displayWith]="displayEmailFn"
                #secondaryPanelAuto="matAutocomplete"
                (optionSelected)="secSelected($event)"
              >
                <mat-option
                  *ngFor="let panel of secondaryPanelAc$ | async"
                  [value]="panel"
                >
                  <!-- <p *ngIf="!checkIfDuplicate(panel)">
                    {{ panel.employeeName }}
                  </p> -->
                  {{ panel.employeeName }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>

            <!-- <mat-form-field>
              <mat-select
                (selectionChange)="onSelect($event, false)"
                name="secondaryEmployeeIds"
                placeholder="Select Candidates"
                formControlName="secondaryEmployeeIds"
                multiple
              >
                <mat-option
                  [title]="employee.emailId"
                  [value]="employee.employeeId"
                  *ngFor="let employee of secondaryPanelMembers"
                  >{{ employee.employeeName }}</mat-option
                >
              </mat-select>
            </mat-form-field> -->
          </div>
        </div>

panel-schedule.component.ts

import { Component, ElementRef, Inject, OnInit, ViewChild } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialog, MatDialogRef} from '@angular/material/dialog';
import { AbstractControl, FormBuilder, FormControl, FormGroup, ValidationErrors,
  Validators} from '@angular/forms';
import { CandidateApplication, ElasticSearchResponse } from '@/src/app/interfaces/app-interface';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import * as moment from 'moment';
import { getTimeZones } from '@vvo/tzdb';
import { MatSelectChange } from '@angular/material/select';
import { MessageModalsComponent } from '@/src/app/common-modules/message-modals/message-modals.component';
import { ResponsiveViewComponent } from '@/src/app/shared/components/responsive-view/responsive-view.component';
import { MediaMatcher } from '@angular/cdk/layout';
import { forEach, uniq } from 'lodash-es';
import { Observable, catchError, of, startWith, debounceTime, switchMap, map } from 'rxjs';
import { MatAutocompleteSelectedEvent } from '@angular/material/autocomplete';

interface PanelMember {
  active: boolean;
  createdDate: string | null;
  emailId: string;
  employeeId: string;
  employeeName: string;
  experience: boolean;
  fresher: boolean;
  id: string;
  jobType: string;
  updatedDate: null;
}

export interface Fruit {
  name: string;
}
@Component({
  selector: 'app-panel-schedule-dialog',
  templateUrl: './panel-schedule-dialog.component.html',
  styleUrls: ['./panel-schedule-dialog.component.scss'],
})
export class PanelScheduleDialogComponent
  extends ResponsiveViewComponent
  implements OnInit
{
  applicationList: {
    applicationId: string;
    email?: string;
    fullName?: string;
    mobile?: string;
  }[] = [];

  min = new Date();
  //timezone = Intl.DateTimeFormat().resolvedOptions().timeZone;

  timingList: any[] = [];
  timeZoneList: any[] = [];
  panelMembers: PanelMember[] = [];
  secondaryPanelMembers: PanelMember[] = [];
  panelForm: FormGroup;
  hrId = '';
  isAvailableTimeChecked: boolean;
  filteredTimeZoneList: Observable<any[]>;
  lastFilter: string = '';
  jobs: {
    jobTitle: string;
    location: string;
    id: string;
    display: string;
    experienceLevel: string;
  }[] = [];
  jobId: string;
  options: any = [];

  get f() {
    return this.panelForm.controls;
  }

  public primaryPanelAc$: any;
  public primaryPanelAcControl = new FormControl<any>(
    { value: '', disabled: false },
    [Validators.required, this.validateEmployeeIdCtrl]
  );
  public secondaryPanelAcControl = new FormControl<any>({
    value: '',
    disabled: false,
  });
  public secondaryPanelAc$: any;

  public jobAc$: any;
  public jobAcControl = new FormControl<any>({ value: '', disabled: false }, [
    Validators.required,
    this.validateJobIdCtrl,
  ]);

  public reqAcControl = new FormControl<any>({ value: '', disabled: false }, [
    // Validators.required,
  ]);
  public reqAc$: any;
  public applicationAcControl = new FormControl<any>({
    value: '',
    disabled: true,
  });
  public applicationAc$: any;

  selectedPanel: PanelMember[] = [];
  selectedCandidates: any[] = [];

  @ViewChild('secAutoCompleteInput')
  secAutoCompleteInput: ElementRef<HTMLInputElement>;

  @ViewChild('candidateAutoCompleteInput')
  candidateAutoCompleteInput: ElementRef<HTMLInputElement>;

  isEditMode: boolean = false;
  dialogHeading: string = 'Panel Schedule';
  scheduleButtonText: string = 'Schedule';
  eventData: any;

  constructor(
    media: MediaMatcher,
    @Inject(MAT_DIALOG_DATA) private data: any,
    private formBuilder: FormBuilder,
    private applicationService: ApplicationService,
    private dialog: MatDialog,
    public dialogRef: MatDialogRef<PanelScheduleDialogComponent>
  ) {
    super(media);
    this.isAvailableTimeChecked = false;
    if (data.job?.jobId) this.jobId = data.job?.jobId;
    if (this.jobId) {
      var reqIdList: any = [];
      this.applicationService.getRequisitionNumber(this.jobId).then((res) => {
        res.forEach((x: { requisitionId: any }) => {
          reqIdList.push(x.requisitionId);
        });
        this.reqAc$ = reqIdList;
      });
    }
    this.hrId = localStorage.getItem('id') as string;
    var timezoneslist = getTimeZones({ includeUtc: true });
    var countryNames = ['Mexico', 'United States', 'Philippines', 'India'];
    var items = timezoneslist.filter((x) =>
      countryNames.includes(x.countryName)
    );
    items.forEach((tz) => {
      timezoneslist.splice(timezoneslist.indexOf(tz), 1);
      if (tz.countryName !== 'India') timezoneslist.unshift(tz);
    });

    timezoneslist.unshift(items?.filter((x) => x.countryName == 'India')[0]);

    this.timeZoneList = timezoneslist;
    this.eventData = data;
  }
  filterMethod(event: any) {
    this.applicationService.getPanelMembersV2(event.query).subscribe((res) => {
      this.options = res;
    });
  }

  onRemoved(applicationId: string) {
    const applicationIds = this.f['applicationIds'].value as string[];
    this.removeFirst(applicationIds, applicationId);
    this.f['applicationIds'].setValue(applicationIds); // To trigger change detection
  }
  private removeFirst<T>(array: T[], toRemove: T): void {
    const index = array.indexOf(toRemove);
    if (index !== -1) {
      array.splice(index, 1);
    }
  }

  remove(employeeId: string): void {
    // remove from selectedPanel
    const index = this.selectedPanel.findIndex(
      (x) => x.employeeId === employeeId
    );

    if (index >= 0) {
      this.selectedPanel.splice(index, 1);
    }
  }
  removec(id: string): void {
    // remove from selectedPanel
    const index = this.selectedCandidates.findIndex((x) => x.id === id);

    if (index >= 0) {
      this.selectedCandidates.splice(index, 1);
    }
  }

  secSelected(event: MatAutocompleteSelectedEvent): void {
    this.selectedPanel.push(event.option.value);
    this.secAutoCompleteInput.nativeElement.value = '';
    this.secondaryPanelAcControl.setValue(null);
  }

  async candidateSelected(event: MatAutocompleteSelectedEvent): Promise<void> {
    if (!this.jobId) {
      this.jobId = event.option.value.jobId;
      var res = await this.applicationService.getJobApplicationByID(
        event.option.value.id
      );
      this.jobLookup(res?.jobTitle).subscribe((res) => {
        var res = res.filter((x: { id: any }) => x.id == this.jobId);
        this.jobAcControl?.setValue(res[0]);
        this.panelForm.patchValue({ jobId: res[0] });
      });
    }
    this.selectedCandidates.push(event.option.value);
    this.candidateAutoCompleteInput.nativeElement.value = '';
    this.applicationAcControl.setValue(null);
  }

  validateEmployeeIdCtrl(ctrl: AbstractControl): ValidationErrors | null {
    const val = ctrl.value;
    if (!val || val === '' || !val?.employeeId) {
      return {
        error: true,
      };
    }
    return null;
  }

  validateJobIdCtrl(ctrl: AbstractControl): ValidationErrors | null {
    const val = ctrl.value;
    if (!val || val === '' || !val?.id) {
      return {
        error: true,
      };
    }
    return null;
  }

  priSelected(event: MatAutocompleteSelectedEvent): void {
    const isAlreadyPresent = this.selectedPanel.find(
      (e) => e.emailId === event.option.value.emailId
    );
    if (isAlreadyPresent) {
      this.f['employeeId'].reset();
      return;
    }
    // this.selectedPanelToCheck.push(event.option.value);
    // this.f['employeeId'].setValue(event.option.value);
  }

  timeZoneChange(event: MatAutocompleteSelectedEvent): void {
    this.min = new Date(
      new Date().toLocaleString('en', { timeZone: event.option.value.name }) //if timezone is changed - show the dates accordingly
    );
    this.checkCal();
  }

  jobSelected(event: MatAutocompleteSelectedEvent): void {
    this.jobId = event.option.value.id;
    if (this.selectedCandidates.length > 0) {
      this.selectedCandidates = this.selectedCandidates.filter(
        (x) => x.jobId == this.jobId
      );
    }
    var reqIdList: any = [];
    this.applicationService.getRequisitionNumber(this.jobId).then((res) => {
      res.forEach((x: { requisitionId: any }) => {
        reqIdList.push(x.requisitionId);
      });
      this.reqAc$ = reqIdList;
    });
  }

  reqSelected(event: MatAutocompleteSelectedEvent): void {
    this.panelForm.get('requisitionId')?.setValue(event.option.value);
  }

  onSelect(event: MatSelectChange, primary: boolean) {
    console.log(event);
    if (primary) {
      this.secondaryPanelMembers = this.panelMembers.filter(
        (e) => e.employeeId !== event.value
      );
    }
    this.checkCal();
  }

  onDateChange(event: any) {
    console.log('on date change', event);
    this.checkCal();
  }
  panelLookup(value: string): Observable<any> {
    const toAvoid = this.selectedPanel.map((e: PanelMember) => e.employeeId);

    return this.applicationService
      .getPanelMembersV2(value?.toLowerCase(), uniq(toAvoid))
      .pipe(
        // map the item property of the results as our return object
        map((results: any) => results),
        // catch errors
        catchError((_) => {
          return of(null);
        })
      );
  }
  
  jobLookup(value: string): Observable<any> {
    return this.applicationService.getAllJobsV2(value?.toLowerCase()).pipe(
      map((results) => {
        console.log(results);
        const res = results.hits.hits
          .map((job: any) => job._source)
          .map((e: any) => {
            return {
              ...e,
              display:
                e.jobTitle +
                ' - ' +
                e.location +
                ' - (' +
                e.experienceLevel +
                ')',
            };
          });
        return res;
      }),
      catchError((_) => {
        return of(null);
      })
    );
  }

  applicationLookup(value: string): Observable<any> {
    const toAvoid = this.selectedCandidates.map((e: any) => e.id);

    return this.applicationService
      .getCandidatesForPanel(value?.toLowerCase(), uniq(toAvoid), this.jobId)
      .pipe(
        // map the item property of the results as our return object
        map((results: any) => results),
        // catch errors
        catchError((_) => {
          return of(null);
        })
      );
  }

  listenForValueChanges() {
    // primary and secondary panel members autocomplete
    this.primaryPanelAc$ = this.primaryPanelAcControl.valueChanges.pipe(
      startWith(''),
      // delay emits
      debounceTime(300),
      // use switch map so as to cancel previous subscribed events, before creating new once
      switchMap((value) => {
        if (value !== '') {
          if (value?.employeeName) {
            return of(null);
          }

          return this.panelLookup(value);
        } else {
          // if no value is present, return null
          return this.panelLookup('');
        }
      })
    );

    this.secondaryPanelAc$ = this.secondaryPanelAcControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      switchMap((value) => {
        if (value !== '') {
          if (value?.employeeName) {
            return of(null);
          }

          return this.panelLookup(value);
        } else {
          // if no value is present, return null
          return this.panelLookup('');
        }
      })
    );

    // job and candidate list autocomplete
    if (this.data?.job?.jobId == null || this.data?.job?.jobId == undefined) {
      this.jobAc$ = this.jobAcControl.valueChanges.pipe(
        startWith(''),
        // delay emits
        debounceTime(300),
        // use switch map so as to cancel previous subscribed events, before creating new once
        switchMap((value) => {
          if (value !== '') {
            if (value?.jobTitle) {
              return of(null);
            }

            return this.jobLookup(value);
          } else {
            // if no value is present, return null
            return this.jobLookup('');
          }
        })
      );
    }

    this.applicationAc$ = this.applicationAcControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      switchMap((value) => {
        if (value !== '') {
          if (value?.name) {
            return of(null);
          }

          return this.applicationLookup(value);
        } else {
          // if no value is present, return null
          return this.applicationLookup('');
        }
      })
    );
  }
  ngOnInit(): void {
    this.initiateForm();
    this.listenForValueChanges();
    console.log('eventData inside dialog', this.eventData?.id);
    if (this.eventData?.id) {
      this.isEditMode = true;
      this.dialogHeading = 'Edit Panel Schedule';
      this.scheduleButtonText = 'Reschedule'
      this.populateFormWithEventData(this.eventData);
    }

    // this.getApplications();
    // this.getPanelMembers();
    this.filteredTimeZoneList = this.panelForm.controls[
      'timezone'
    ].valueChanges.pipe(
      startWith<string | any[]>(''),
      map((value) => (typeof value === 'string' ? value : this.lastFilter)),
      map((filter) => this.filter(filter))
    );
  }

  populateFormWithEventData(eventData: any): void {
    // Set the Job ID (Job Title)
    this.panelForm.patchValue({
      jobId: {
        id: eventData.jobId,
        jobTitle: eventData.title,
        oracleJobReference: eventData.oracleJobReference,
        location: eventData.location,
        display: eventData.display,
      },
    });

    this.panelForm.patchValue({
      requisitonId: eventData.id,
    });

     

    if (eventData.candidates && eventData.candidates.length > 0) {
      console.log("EVENT CANDIDATES", eventData.candidates);
      //this.selectedCandidates = eventData.candidates.filter((candidate: any) => candidate.status == null || candidate.status == undefined);
      this.selectedCandidates = eventData.candidates
        this.applicationAcControl.setValue(
            this.selectedCandidates.map((candidate: any) => candidate.applicationId)
        );
    }

    // Set Primary Panel Member
    const primaryPanelMember = eventData.panelMembers.find(
      (pm: any) => pm.primaryPanelMember === true
    );
    if (primaryPanelMember) {
      const primaryPanelMemberValue = {
        id: primaryPanelMember.employeeId,
        employeeId: primaryPanelMember.employeeId,
        emailId: primaryPanelMember.email,
        employeeName: primaryPanelMember.name,
      };
      this.panelForm.patchValue({
        employeeId: primaryPanelMemberValue,
      });
    }

    // Set Secondary Panel Members
    const secondaryPanelMembers = eventData.panelMembers.filter(
      (pm: any) => pm.primaryPanelMember === false
    );
    const secondaryPanelMemberIds = secondaryPanelMembers.map(
      (sec: any) => sec.employeeId
    );
   

    this.panelForm.patchValue({
      secondaryEmployeeIds: secondaryPanelMemberIds,
    });

    // Set Date and Time
    const startDate = new Date(eventData.start);
    this.panelForm.patchValue({
      date: startDate,
      timings: {
        startTime: new Date(eventData.start),
        endTime: new Date(eventData.end),
      },
    });

    // Set Timezone
    this.panelForm.patchValue({
      timezone: {
        name: eventData.timezone.name,
        abbreviation: eventData.timezone.abbreviation,
        mainCities: eventData.timezone.mainCities,
      },
    });
  }

  filter(filter: string): any[] {
    this.lastFilter = filter;
    if (filter) {
      return this.timeZoneList.filter((option) => {
        return (
          option.name.toLowerCase().indexOf(filter.toLowerCase()) >= 0 ||
          option.countryName.toLowerCase().indexOf(filter.toLowerCase()) >= 0 ||
          option.abbreviation.toLowerCase().indexOf(filter.toLowerCase()) >=
            0 ||
          option.currentTimeFormat
            .toLowerCase()
            .indexOf(filter.toLowerCase()) >= 0
        );
      });
    } else {
      return this.timeZoneList.slice();
    }
  }

  displayFn(value: any): any {
    return `(UTC${value.currentTimeFormat.substring(
      0,
      6
    )}) - ${value.mainCities.join(', ')}`;
    // this.getJobs();
  }

  checkCal() {
    console.log(this.f);
    const panelMembers = [
      this.f['employeeId'].value?.emailId,
      ...this.selectedPanel.map((e) => e.emailId),
    ].filter((e) => e);
    if (this.f['date'].value && panelMembers.length > 0) {
      console.log(panelMembers);
      const payload = {
        interviewDate: moment(this.f['date'].value).format('YYYY-MM-DD'),
        timeZone:
          this.panelForm.value.timezone.name === 'Asia/Calcutta'
            ? 'Asia/Kolkata'
            : this.panelForm.value.timezone.name,
        emailIds: uniq(panelMembers),
      };
      console.log(payload);
      this.applicationService.checkAvailability(payload).then((res) => {
        this.isAvailableTimeChecked = true;
        this.timingList = res[0].availableTimes.map((e: any) => ({
          startTime: e?.['startTime'],
          endTime: e?.['endTime'],
        }));
      });
    }
  }

  getJobs() {
    this.applicationService
      .getAllJobs({
        orderBy: 'desc',
        limit: 500,
        offset: 0,
      })
      .then((res) => {
        this.jobs = res.hits.hits.map((job: any) => {
          const j = job._source;
          return {
            id: j.id,
            location: j.location,
            jobTitle: j.jobTitle,
            experienceLevel: j.experienceLevel,
            display:
              j.jobTitle +
              ' - ' +
              j.location +
              ' - (' +
              j.experienceLevel +
              ')',
          };
        });
      });
  }

  displayEmailFn(
    option: { emailId: string; employeeId: string; employeeName: string } | null
  ) {
    return option ? option.employeeName : '';
  }

  displayJobFn(
    option: {
      display: string;
      jobTitle: string;
      location: string;
      experienceLevel: string;
    } | null
  ) {
    return option ? option.display : '';
  }

  displayFullNameFn(
    option: {
      name: string;
    } | null
  ) {
    return option ? option.name : '';
  }

  initiateForm() {
    this.panelForm = this.formBuilder.group({
      jobId: this.jobAcControl,
      requisitonId: this.reqAcControl,
      applicationIds: this.applicationAcControl,
      employeeId: this.primaryPanelAcControl,
      secondaryEmployeeIds: this.secondaryPanelAcControl,
      date: ['', [Validators.required]],
      timings: [{ value: null, disabled: true }, [Validators.required]],
      timezone: ['', [Validators.required]],
    });

    this.panelForm.get('jobId')?.valueChanges.subscribe((value) => {
      if (value) {
        this.f['applicationIds'].reset();
        this.f['applicationIds'].enable();
      }
    });
    this.panelForm.valueChanges.subscribe((value) => {
      console.log('e:> ', this.f['employeeId']);
      if (
        value['date'] &&
        (value['employeeId'] || value['secondaryEmployeeIds']?.length > 0)
      ) {
        this.f['timings']?.enable({ emitEvent: false });
      } else {
        this.f['timings']?.disable({ emitEvent: false });
      }
    });

    if (this.data?.candidates?.length > 0) {
      this.applicationLookup(this.data?.candidates[0]?.fullName).subscribe(
        (res) => {
          this.applicationAcControl.setValue(res[0]);
          this.selectedCandidates.push(res[0]);
        }
      );

      this.panelForm.patchValue({
        applicationIds: this.data.candidates.map((e: any) => e.applicationId),
      });
    }
    if (this.data?.job?.jobId) {
      this.jobLookup(this.data?.job?.jobTitle).subscribe((res) => {
        var res = res.filter((x: { id: any }) => x.id == this.data.job.jobId);
        this.jobAcControl?.setValue(res[0]);
        this.panelForm.patchValue({ jobId: res[0] });
      });
      this.panelForm.get('jobId')?.disable();
    }
  }

  public weekendFilter = (d: any): boolean => {
    const day = d.weekday();
    // Prevent Saturday and Sunday from being selected.
    return day !== 0 && day !== 6;
  };

  transformList(
    response: ElasticSearchResponse<CandidateApplication>
  ): CandidateApplication[] {
    return response.hits.hits.map((job) => {
      return job._source;
    });
  }


  onSubmit() {
    console.log('Form value', this.panelForm?.value);
    console.log('job', this.jobAcControl.value);
    const payload = {
      applicationIds: this.selectedCandidates.map((e) => e.id),
      ...this.panelForm.value.timings,
      timeZone:
        this.panelForm.value.timezone.name === 'Asia/Calcutta'
          ? 'Asia/Kolkata'
          : this.panelForm.value.timezone.name,
      employeeId: this.panelForm.value.employeeId.employeeId,
      secondaryEmployeeIds: this.selectedPanel
        .map((e: PanelMember) => e.employeeId)
        .filter((e) => e !== this.panelForm.value.employeeId.employeeId),
      hrId: this.hrId,
      jobId: this.jobId,
      requisitionId: this.panelForm.value.requisitonId,
    };

    console.log('Our Payload', payload.secondaryEmployeeIds);

    // this.applicationService.schedulePanel(payload).then((_res) => {
    //   this.dialog.open(MessageModalsComponent, {
    //     maxWidth: this.mobileView ? '500px' : '100%',
    //     width: this.mobileView ? '100%' : '30%',
    //     ...(this.mobileView && {
    //       height: 'auto',
    //       position: { bottom: '0' },
    //     }),
    //     closeOnNavigation: true,
    //     disableClose: true,
    //     data: 'Panel Scheduled,statusChange',
    //   });
    //   this.dialogRef.close(1);
    // });
  }

  closeModal() {
    this.dialogRef.close();
  }
}


eventdata panelmembers setails

[
    {
        "id": "9a9b99d4-f585-4939-9cc5-1c2dba28d670",
        "primaryPanelMember": true,
        "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
        "name": "Kabildev Chellamuthu",
        "email": "kabildev.chellamuthu@agshealth.com",
        "mobile": "9600795023"
    },
    {
        "id": "786ba3ba-edd2-4bfb-b834-a6463b8fdc6e",
        "primaryPanelMember": false,
        "employeeId": "5f7bfea9-b7c3-4afc-8885-621c3adf5275",
        "name": "Vishnu Manojkumar",
        "email": "vishnu.manojkumar@agshealth.com",
        "mobile": "8220210765"
    }
]





