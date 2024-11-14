panel-schedule-dialog.component.ts

import {
  Component,
  ElementRef,
  Inject,
  OnInit,
  ViewChild,
} from '@angular/core';
import {
  MAT_DIALOG_DATA,
  MatDialog,
  MatDialogRef,
} from '@angular/material/dialog';
import {
  AbstractControl,
  FormBuilder,
  FormControl,
  FormGroup,
  ValidationErrors,
  Validators,
} from '@angular/forms';
import {
  CandidateApplication,
  ElasticSearchResponse,
} from '@/src/app/interfaces/app-interface';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import * as moment from 'moment';
import { getTimeZones } from '@vvo/tzdb';
import { MatSelectChange } from '@angular/material/select';
import { MessageModalsComponent } from '@/src/app/common-modules/message-modals/message-modals.component';
import { ResponsiveViewComponent } from '@/src/app/shared/components/responsive-view/responsive-view.component';
import { MediaMatcher } from '@angular/cdk/layout';
import { forEach, uniq } from 'lodash-es';
import {
  Observable,
  catchError,
  of,
  startWith,
  debounceTime,
  switchMap,
  map,
} from 'rxjs';
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
    Validators.required,
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
  // .hits.hits.map((job) => {
  //       return job._source;
  //     })
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
          return this.applicationLookup('');
        }
      })
    );
  }
  ngOnInit(): void {
    this.initiateForm();
    this.listenForValueChanges();
    console.log('eventData inside dialog', this.eventData);
    if (this.eventData.id) {
      this.isEditMode = true;
      this.dialogHeading = 'Edit Panel Schedule';
      this.scheduleButtonText = 'Reschedule';
      this.populateFormWithEventData();
       console.log('Selected Panels', this.selectedPanel);
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
    console.log("Form value", this.panelForm.value);
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

    console.log(payload);

    this.applicationService.schedulePanel(payload).then((_res) => {
      this.dialog.open(MessageModalsComponent, {
        maxWidth: this.mobileView ? '500px' : '100%',
        width: this.mobileView ? '100%' : '30%',
        ...(this.mobileView && {
          height: 'auto',
          position: { bottom: '0' },
        }),
        closeOnNavigation: true,
        disableClose: true,
        data: 'Panel Scheduled,statusChange',
      });
      this.dialogRef.close(1);
    });
  }

  closeModal() {
    this.dialogRef.close();
  }
}

panel-schedule-dialog.component.html

import {
  Component,
  ElementRef,
  Inject,
  OnInit,
  ViewChild,
} from '@angular/core';
import {
  MAT_DIALOG_DATA,
  MatDialog,
  MatDialogRef,
} from '@angular/material/dialog';
import {
  AbstractControl,
  FormBuilder,
  FormControl,
  FormGroup,
  ValidationErrors,
  Validators,
} from '@angular/forms';
import {
  CandidateApplication,
  ElasticSearchResponse,
} from '@/src/app/interfaces/app-interface';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import * as moment from 'moment';
import { getTimeZones } from '@vvo/tzdb';
import { MatSelectChange } from '@angular/material/select';
import { MessageModalsComponent } from '@/src/app/common-modules/message-modals/message-modals.component';
import { ResponsiveViewComponent } from '@/src/app/shared/components/responsive-view/responsive-view.component';
import { MediaMatcher } from '@angular/cdk/layout';
import { forEach, uniq } from 'lodash-es';
import {
  Observable,
  catchError,
  of,
  startWith,
  debounceTime,
  switchMap,
  map,
} from 'rxjs';
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
    Validators.required,
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
  // .hits.hits.map((job) => {
  //       return job._source;
  //     })
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
      // map the item property of the results as our return object
      // map((results: any) => ({
      //   ...results,
      //   display:
      //     results.jobTitle +
      //     ' - ' +
      //     results.location +
      //     ' - (' +
      //     results.experienceLevel +
      //     ')',
      // })),
      // catch errors
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

  // checkIfDuplicate(item: PanelMember) {
  //   return this.selectedPanel.some(
  //     (e: PanelMember) => e.employeeId === item.employeeId
  //   );
  // }

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
    console.log('eventData inside dialog', this.eventData);
    if (this.eventData.id) {
      this.isEditMode = true;
      this.dialogHeading = 'Edit Panel Schedule';
      this.scheduleButtonText = 'Reschedule';
      this.populateFormWithEventData();
       console.log('Selected Panels', this.selectedPanel);
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

  populateFormWithEventData() {
    if (this.eventData) {
      const { panelMembers, candidates, start, end, title } = this.eventData;

      this.panelForm.patchValue({
        date: moment(start).format('YYYY-MM-DD'),
        timings: {
          startTime: moment(start).format('HH:mm'),
          endTime: moment(end).format('HH:mm'),
        },
        timezone: "Asia/Kolkata",
      });

      if (candidates && candidates.length > 0) {
        this.selectedCandidates = candidates;
        this.applicationAcControl.setValue(
          candidates.map((candidate: any) => candidate.applicationId)
        );
      }

      if (panelMembers && panelMembers.length > 0) {
        const primaryMember = panelMembers.find(
          (member: any) => member.primaryPanelMember
        );
        const secondaryMembers = panelMembers.find(
          (member: any) => !member.primaryPanelMember
        );

        console.log("Primary Members", primaryMember);
        console.log("Secondary Members", secondaryMembers);

        if (primaryMember) {
          this.primaryPanelAcControl.setValue(primaryMember.id);
          this.selectedPanel.push(primaryMember);
        }
        
        if (secondaryMembers) {
          this.secondaryPanelAcControl.setValue(
            secondaryMembers.map((member: any) => member.employeeId)
          );
          this.selectedPanel.push(...secondaryMembers);
        }

      }

      if (title) {
        this.jobAcControl.setValue(title);
        this.panelForm.patchValue({ jobId: title });
      }

      
    }
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

  // getApplications() {
  //   const applicationIds = this.f['applicationIds'].value;
  //   this.applicationService
  //     .getCandidatesForPanel(applicationIds ? applicationIds : [], this.jobId)
  //     .then((response) => {
  //       console.log(response);
  //       this.applicationList = response.map((e) => {
  //         return {
  //           email: e.email,
  //           fullName: e?.name ? e.name : e.email.split('@')[0],
  //           applicationId: e.id,
  //         };
  //       });
  //     });
  // }

  onSubmit() {
    console.log("Form value", this.panelForm.value);
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

    console.log(payload);

    this.applicationService.schedulePanel(payload).then((_res) => {
      this.dialog.open(MessageModalsComponent, {
        maxWidth: this.mobileView ? '500px' : '100%',
        width: this.mobileView ? '100%' : '30%',
        ...(this.mobileView && {
          height: 'auto',
          position: { bottom: '0' },
        }),
        closeOnNavigation: true,
        disableClose: true,
        data: 'Panel Scheduled,statusChange',
      });
      this.dialogRef.close(1);
    });
  }

  closeModal() {
    this.dialogRef.close();
  }
}

EventData

{
    "id": "bcef7f87-b555-4038-9dea-5b111eafef65",
    "start": "2024-11-24T01:30:00.000Z",
    "end": "2024-11-24T02:00:00.000Z",
    "panelMembers": [
        {
            "id": "ff329c2d-597d-4aed-bb3c-858960dfaa3b",
            "primaryPanelMember": false,
            "employeeId": "5f7bfea9-b7c3-4afc-8885-621c3adf5275",
            "name": "Vishnu Manojkumar",
            "email": "vishnu.manojkumar@agshealth.com",
            "mobile": "8220210765"
        },
        {
            "id": "299e96ce-62a5-4d5d-8aa7-5ead51bc92d3",
            "primaryPanelMember": true,
            "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
            "name": "Kabildev Chellamuthu",
            "email": "kabildev.chellamuthu@agshealth.com",
            "mobile": "9600795023"
        }
    ],
    "candidates": [
        {
            "id": "ad14df4c-a52e-4694-a2bf-bf6dc6d02eb1",
            "candidateId": "bb188c98-0031-4b1b-94ac-9caa461dbd50",
            "name": "Abc jrnvbn",
            "email": "abc234@abc.com",
            "mobile": "9098674999",
            "applicationId": "c16d9b5f-0cdd-463b-8ac6-1cf569c404b3",
            "applicationStatus": null,
            "interviewStatus": "PROGRESS",
            "active": true,
            "feedback": null,
            "panelUpdatedEmployee": null
        },
        {
            "id": "13a45a09-93b5-47c2-b4c1-ce45fd4dea87",
            "name": "Test Voice",
            "email": "abc456789@abc.com",
            "userId": "fac5eb66-635a-4c56-9c73-7a9733fd15a0",
            "jobId": "e5d2e945-6b57-42ae-90f8-37651b5996c2",
            "status": "PANEL_YET_TO_SCHEDULE",
            "mobile": "9098765657"
        }
    ],
    "title": "Interview | Voice Experienced Pb",
    "color": {
        "primary": "#2b3874",
        "secondary": "#fafaff"
    },
    "actions": [
        {
            "label": "https://teams.microsoft.com/l/meetup-join/19%3ameeting_NjkzMWE5ZDMtOGI0ZC00ZTZmLWI1YTctMjkzZjdmNTBkNWM5%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%229c344eb3-53a2-45af-9f02-f764bc2f2e49%22%7d"
        }
    ],
    "allDay": false,
    "resizable": {
        "beforeStart": false,
        "afterEnd": false
    },
    "draggable": false,
    "meta": {
        "meetingLink": "https://teams.microsoft.com/l/meetup-join/19%3ameeting_NjkzMWE5ZDMtOGI0ZC00ZTZmLWI1YTctMjkzZjdmNTBkNWM5%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%229c344eb3-53a2-45af-9f02-f764bc2f2e49%22%7d",
        "interviewType": null,
        "hrId": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "panelMembers": [
            {
                "id": "ff329c2d-597d-4aed-bb3c-858960dfaa3b",
                "primaryPanelMember": false,
                "employeeId": "5f7bfea9-b7c3-4afc-8885-621c3adf5275",
                "name": "Vishnu Manojkumar",
                "email": "vishnu.manojkumar@agshealth.com",
                "mobile": "8220210765"
            },
            {
                "id": "299e96ce-62a5-4d5d-8aa7-5ead51bc92d3",
                "primaryPanelMember": true,
                "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
                "name": "Kabildev Chellamuthu",
                "email": "kabildev.chellamuthu@agshealth.com",
                "mobile": "9600795023"
            }
        ],
        "candidates": [
            {
                "id": "ad14df4c-a52e-4694-a2bf-bf6dc6d02eb1",
                "candidateId": "bb188c98-0031-4b1b-94ac-9caa461dbd50",
                "name": "Abc jrnvbn",
                "email": "abc234@abc.com",
                "mobile": "9098674999",
                "applicationId": "c16d9b5f-0cdd-463b-8ac6-1cf569c404b3",
                "applicationStatus": null,
                "interviewStatus": "PROGRESS",
                "active": true,
                "feedback": null,
                "panelUpdatedEmployee": null
            },
            {
                "id": "13a45a09-93b5-47c2-b4c1-ce45fd4dea87",
                "name": "Test Voice",
                "email": "abc456789@abc.com",
                "userId": "fac5eb66-635a-4c56-9c73-7a9733fd15a0",
                "jobId": "e5d2e945-6b57-42ae-90f8-37651b5996c2",
                "status": "PANEL_YET_TO_SCHEDULE",
                "mobile": "9098765657"
            }
        ],
        "eventId": "AAMkADAzOTM5MDM5LWU1NDUtNDcxZS04NGNkLTQ5OTI4YmNhZjNiNQBGAAAAAADclWry2I3nSqkjw7GWRnvVBwA1uIZfWZT2QrlhZYeipAE7AAAAAAENAAA1uIZfWZT2QrlhZYeipAE7AAC5ICRbAAA="
    }
}



Example form values after submission


{
    "jobId": {
        "id": "e5d2e945-6b57-42ae-90f8-37651b5996c2",
        "jobTitle": "Voice - Experience - HB - Chennai",
        "oracleJobReference": "445",
        "lineOfBusiness": "ORA_PROFESSIONAL",
        "location": "Chennai",
        "shift": "NIGHT_SHIFT",
        "shortDesc": "Minimum 1 - 5 Years of Experience in Denial Management, Immediate Joiners Preferred",
        "longDesc": "<p><strong>ROLE SUMMARY</strong></p>\n<p>&nbsp;</p>\n<p>This position is based out of the Philippines- Manila office, responsible for the following:&nbsp;</p>\n<p>&nbsp;</p>\n<ul>\n <li>&nbsp;Ensure quality driven follow-up activities and resolution of accounts is carried out with the insurance carriers on the outstanding inventory to yield maximum cash flow and minimum bad debts</li>\n <li>Interact by Phone and Internet based portals with the insurance companies in US to procure status of the claim followed by appropriate activities to address open AR</li>\n <li>The key functions of an AR Executive are to ensure the below responsibilities are carried out to the best of his/her ability in the interest of the organization and client</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>RESPONSIBILITIES</strong></p>\n<p>&nbsp;</p>\n<ul>\n <li>To address outstanding or assigned AR through analysis and phone calls by using available resources</li>\n <li>Utilization of all possible tools and applications available to take account to the next level of resolution, which would result in a payment, corrected submission, appeals, patient transfer or adjustment</li>\n <li>To report trends / patterns in denials, claim submission errors, credentialing issues and billing related roadblocks to the immediate reporting manager.</li>\n <li>To meet the established SLAs (service level agreements) for production and quality</li>\n <li>To update the outcome of the calls or analysis in a clear and coherent manner in the billing system</li>\n <li>To utilize the P &amp; P’s (policies and procedures) established for the process and stay updated with changes done with the P &amp; Ps</li>\n <li>To improve the performance based on the feedback provided by the reporting manager / quality audit team</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>COMPLIANCE</strong></p>\n<p>Awareness and adherence to all&nbsp;applicable organization wide policies and procedures, including but not limited to Information security, HIPAA, and HR policies.</p>\n<p>&nbsp;</p>\n<p><strong>ACADEMIC AND PROFESSIONAL BACKGROUND</strong></p>\n<ul>\n <li>University graduate with an average aptitude score</li>\n <li>Minimum 12 months – 4 years of AR follow-up experience with either physician or hospital billing background</li>\n <li>CPAT/CCAT certification will be an added advantage</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>COMPETENCIES, SKILLS, AND OTHER REQUISITES</strong></p>\n<p>&nbsp;</p>\n<ul>\n <li>Sound analytical skills</li>\n <li>Logical thinking</li>\n <li>Attention to detail</li>\n <li>Adequate domain knowledge on various billing terminologies &amp; processes</li>\n <li>Good verbal and written communication skills</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>COMPENSATION AND BENEFITS -&nbsp;</strong>Competitive remuneration + annual performance-based bonus + standard industry benefits. Relocation package, per company policy will apply for candidates outside the office base city.&nbsp;</p>\n<p>&nbsp;</p>\n<p><strong>ASSESSMENT / INTERVIEW PROCESS</strong></p>\n<p>&nbsp;</p>\n<ol>\n <li>HR Interview</li>\n <li>Domain assessment&nbsp;</li>\n <li>Final Interview (Domain interview by Op’s)&nbsp;</li>\n</ol>",
        "openPositions": 100,
        "postedAt": 1674040789779,
        "visibility": true,
        "jobType": "CONTRACT",
        "processType": "VOICE",
        "jobHours": "FULL_TIME",
        "experienceLevel": "Experienced",
        "cashBonus": 0,
        "oracleJobReferenceNumber": "300000129588287",
        "lob": "HB",
        "blockApplyButton": false,
        "coolingPeriod": false,
        "deleted": false,
        "display": "Voice - Experience - HB - Chennai - Chennai - (Experienced)"
    },
    "requisitonId": "1234",
    "applicationIds": null,
    "employeeId": {
        "id": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
        "createdDate": null,
        "updatedDate": null,
        "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
        "jobType": "VOICE",
        "experience": true,
        "emailId": "kabildev.chellamuthu@agshealth.com",
        "employeeName": "Kabildev Chellamuthu",
        "fresher": true,
        "active": true
    },
    "secondaryEmployeeIds": null,
    "date": "2024-11-27T18:30:00.000Z",
    "timings": {
        "startTime": "2024-11-28T12:30:00",
        "endTime": "2024-11-28T13:00:00"
    },
    "timezone": {
        "name": "Asia/Kolkata",
        "alternativeName": "India Time",
        "group": [
            "Asia/Kolkata",
            "Asia/Calcutta"
        ],
        "continentCode": "AS",
        "continentName": "Asia",
        "countryName": "India",
        "countryCode": "IN",
        "mainCities": [
            "Mumbai",
            "Delhi",
            "Bengaluru",
            "Hyderābād"
        ],
        "rawOffsetInMinutes": 330,
        "abbreviation": "IST",
        "rawFormat": "+05:30 India Time - Mumbai, Delhi, Bengaluru, Hyderābād",
        "currentTimeOffsetInMinutes": 330,
        "currentTimeFormat": "+05:30 India Time - Mumbai, Delhi, Bengaluru, Hyderābād"
    }
}









