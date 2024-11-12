The below is open edit event in angular which calls panel schedule dialog box with event data.Now fill in the panelMembers(primary and secondary from  "primaryPanelMember" value true/false),candidates,Date(from start and end value), scheduleMeeting(from start timing to end timing) and if data contains eventId change the dialog heading to Edit Panel Schedule and schedule button to Reschedule

event data

{
    "id": "f6675c2d-727c-47d9-be13-6c3c3498c477",
    "start": "2024-11-19T01:00:00.000Z",
    "end": "2024-11-19T01:30:00.000Z",
    "panelMembers": [
        {
            "id": "bacfa10c-6289-4a63-b3ac-92365b34358c",
            "primaryPanelMember": true,
            "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
            "name": "Kabildev Chellamuthu",
            "email": "kabildev.chellamuthu@agshealth.com",
            "mobile": "9600795023"
        },
          {
            "id": "bacfa10c-6289-4a63-b3ac-92365b34358c",
            "primaryPanelMember": false,
            "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
            "name": "Vishnu",
            "email": "kabildev.chellamuthu@agshealth.com",
            "mobile": "9600795023"
        }
    ],
    "candidates": [
        {
            "id": "77e98dba-6132-47b9-94d8-cedd35b16644",
            "candidateId": "b9e4375f-1816-4040-8634-6395365ca32b",
            "name": "Abc ktik",
            "email": "abc@gmail.com",
            "mobile": "9876543210",
            "applicationId": "878e5247-5a7d-4af6-bf75-2bb8eff07137",
            "applicationStatus": null,
            "interviewStatus": "PROGRESS",
            "active": true,
            "feedback": null,
            "panelUpdatedEmployee": null
        }
    ],
    "title": "Interview | Voice Experienced Hb",
    "color": {
        "primary": "#2b3874",
        "secondary": "#fafaff"
    },
    "actions": [
        {
            "label": "https://teams.microsoft.com/l/meetup-join/19%3ameeting_MTcwMjNmNjItODE3NC00NTJhLWFlZDQtM2E4N2VlNThlOGE2%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%229c344eb3-53a2-45af-9f02-f764bc2f2e49%22%7d"
        }
    ],
    "allDay": false,
    "resizable": {
        "beforeStart": false,
        "afterEnd": false
    },
    "draggable": false,
    "meta": {
        "meetingLink": "https://teams.microsoft.com/l/meetup-join/19%3ameeting_MTcwMjNmNjItODE3NC00NTJhLWFlZDQtM2E4N2VlNThlOGE2%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%229c344eb3-53a2-45af-9f02-f764bc2f2e49%22%7d",
        "interviewType": null,
        "hrId": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "panelMembers": [
            {
                "id": "bacfa10c-6289-4a63-b3ac-92365b34358c",
                "primaryPanelMember": true,
                "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
                "name": "Kabildev Chellamuthu",
                "email": "kabildev.chellamuthu@agshealth.com",
                "mobile": "9600795023"
            }
        ],
        "candidates": [
            {
                "id": "77e98dba-6132-47b9-94d8-cedd35b16644",
                "candidateId": "b9e4375f-1816-4040-8634-6395365ca32b",
                "name": "Abc ktik",
                "email": "abc@gmail.com",
                "mobile": "9876543210",
                "applicationId": "878e5247-5a7d-4af6-bf75-2bb8eff07137",
                "applicationStatus": null,
                "interviewStatus": "PROGRESS",
                "active": true,
                "feedback": null,
                "panelUpdatedEmployee": null
            }
        ],
        "eventId": "AAMkADAzOTM5MDM5LWU1NDUtNDcxZS04NGNkLTQ5OTI4YmNhZjNiNQBGAAAAAADclWry2I3nSqkjw7GWRnvVBwA1uIZfWZT2QrlhZYeipAE7AAAAAAENAAA1uIZfWZT2QrlhZYeipAE7AAC4bgsSAAA="
    }
}

openEditEvent method 

openEditEvent() {
    console.log("Data inside event", this.event);
    this.dialog.open(PanelScheduleDialogComponent, {
      maxWidth: this.mobileView ? '100%' : '100%',
      width: this.mobileView ? '100%' : '654px',
      height: 'auto',
      ...(this.mobileView && { height: 'auto', position: { bottom: '0' } }),
      closeOnNavigation: true,
      disableClose: false,
      autoFocus: false,
      panelClass: 'custom-dialog-container',
      data: this.event,
    });
  }
  declineEvent(eventId: any) {}
}

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

  constructor(
    media: MediaMatcher,
    @Inject(MAT_DIALOG_DATA) private data: { candidates: any[]; job: any },
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
        applicationIds: this.data.candidates.map((e) => e.applicationId),
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


panel-schdule.component.html

<div class="upload-candidate-modal">
  <div class="upload-candidate-header">
    <h5 class="heading-text">Panel Schedule</h5>
    <div>
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>

      <!-- <img
        aria-hidden="true"
        src="assets/images/close-icon.svg"
        alt=""
        (click)="closeModal()"
      /> -->
    </div>
  </div>
  <form [formGroup]="panelForm" (ngSubmit)="onSubmit()">
    <div class="upload-candidate-body">
      <div class="row">
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Job<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select placeholder="Select Job" formControlName="jobId">
                <mat-option
                  [title]="job.jobTitle"
                  *ngFor="let job of jobs"
                  [value]="job.id"
                  >{{ job.display }}</mat-option
                >
              </mat-select> -->
              <input
                [formControl]="jobAcControl"
                type="text"
                placeholder="search jobs"
                aria-label="string"
                matInput
                [matAutocomplete]="jobAutoComplete"
              />
              <mat-autocomplete
                [displayWith]="displayJobFn"
                autoActiveFirstOption
                #jobAutoComplete="matAutocomplete"
                (optionSelected)="jobSelected($event)"
              >
                <mat-option
                  *ngFor="let item of jobAc$ | async; let index = index"
                  [value]="item"
                >
                  {{ item.display | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Requisition Number<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select placeholder="Select Job" formControlName="jobId">
                <mat-option
                  [title]="job.jobTitle"
                  *ngFor="let job of jobs"
                  [value]="job.id"
                  >{{ job.display }}</mat-option
                >
              </mat-select> -->
              <input
                [formControl]="reqAcControl"
                type="text"
                placeholder="search requisition number"
                aria-label="string"
                matInput
                [matAutocomplete]="reqAutoComplete"
              />
              <mat-autocomplete
                autoActiveFirstOption
                #reqAutoComplete="matAutocomplete"
                (optionSelected)="reqSelected($event)"
              >
                <mat-option
                  *ngFor="let item of reqAc$"
                  [value]="item"
                >
                  {{ item }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Candidates<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select
                placeholder="Select Candidates"
                formControlName="applicationIds"
                multiple
              >
                <mat-option
                  [title]="application.email"
                  *ngFor="let application of applicationList"
                  [value]="application.applicationId"
                  >{{ application.fullName }}{{application.mobile}}</mat-option
                >
              </mat-select> -->
              <mat-chip-grid
                #candidateChipGrid
                aria-label="Candidate selection"
              >
                <mat-chip-row
                  *ngFor="let item of selectedCandidates"
                  (removed)="removec(item.id)"
                >
                  {{ item.name }} ({{ item.mobile }})
                  <button matChipRemove [attr.aria-label]="'remove ' + item">
                    <app-icon icon="close_small"></app-icon>
                  </button>
                </mat-chip-row>
              </mat-chip-grid>
              <input
                placeholder="Search candidates"
                #candidateAutoCompleteInput
                [formControl]="applicationAcControl"
                [matChipInputFor]="candidateChipGrid"
                [matAutocomplete]="candidateAuto"
              />
              <mat-autocomplete
                [displayWith]="displayFullNameFn"
                #candidateAuto="matAutocomplete"
                (optionSelected)="candidateSelected($event)"
              >
                <mat-option
                  *ngFor="let application of applicationAc$ | async"
                  [value]="application"
                >
                  <!-- <p *ngIf="!checkIfDuplicate(panel)">
                    {{ panel.employeeName }}
                  </p> -->
                  {{ application.name }} ({{ application.mobile }})
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-inner">
            <label class="form-label" for="employeeId">
              Panel member<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select
                (selectionChange)="onSelect($event, true)"
                placeholder="Select panel member"
                name="employeeId"
                multiple="false"
                formControlName="employeeId"
              >
                <mat-option
                  [title]="employee.emailId"
                  [value]="employee.employeeId"
                  *ngFor="let employee of panelMembers"
                  >{{ employee.employeeName }}</mat-option
                >
              </mat-select> -->

              <input
                [formControl]="primaryPanelAcControl"
                type="text"
                placeholder="search primary panel member"
                aria-label="string"
                matInput
                [matAutocomplete]="primaryAutoComplete"
              />
              <mat-autocomplete
                [displayWith]="displayEmailFn"
                autoActiveFirstOption
                #primaryAutoComplete="matAutocomplete"
                (optionSelected)="priSelected($event)"
              >
                <mat-option
                  *ngFor="
                    let item of primaryPanelAc$ | async;
                    let index = index
                  "
                  [value]="item"
                >
                  {{ item.employeeName | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
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
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="email" class="form-label"
              >Time Zone<span class="required"></span
            ></label>
            <mat-form-field>
              <input
                type="text"
                placeholder="Select Timezone"
                matInput
                [matAutocomplete]="auto"
                formControlName="timezone"
              />
              <mat-autocomplete
                #auto="matAutocomplete"
                [displayWith]="displayFn"
                name="timezone"
                (optionSelected)="timeZoneChange($event)"
              >
                <mat-option
                  class="time-select-panel"
                  *ngFor="let item of filteredTimeZoneList | async"
                  [value]="item"
                >
                  {{
                    '(UTC' +
                      item.currentTimeFormat.substring(0, 6) +
                      ') - ' +
                      item.mainCities.join(', ')
                  }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
      </div>
      <div class="row d-flex align-items-center">
        <div class="col-lg-6 col-sm-12">
          <div class="form-group form-inner">
            <label for="email" class="form-label"
              >Date<span class="required"></span
            ></label>
            <div class="form-datepicker">
              <input
                (dateChange)="onDateChange($event)"
                [min]="min"
                placeholder="Select Date"
                formControlName="date"
                readonly
                class="form-control"
                [matDatepicker]="picker"
              />
              <mat-datepicker-toggle
                matIconSuffix
                [for]="picker"
              ></mat-datepicker-toggle>
            </div>
            <mat-datepicker #picker></mat-datepicker>
            <div
              class="form-invalid"
              *ngIf="timingList.length < 1 && isAvailableTimeChecked"
            >
              <p>Not available for the given date</p>
            </div>
          </div>
        </div>
        <div class="col-lg-6 col-sm-12">
          <div class="">
            <div class="form-inner">
              <label class="form-label" for="employeeId">
                Allocate schedule<span class="required"></span
              ></label>
              <mat-form-field>
                <mat-select
                  placeholder="Schedule"
                  name="employeeId"
                  multiple="false"
                  formControlName="timings"
                >
                  <mat-option *ngFor="let option of timingList" [value]="option"
                    ><span class="mat-radio-label"
                      >{{ option['startTime'] | date : 'hh:mm a' }} -
                      {{ option['endTime'] | date : 'hh:mm a' }}</span
                    ></mat-option
                  >
                </mat-select>
              </mat-form-field>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="upload-candidate-footer">
      <div>
        <button
          title="Cancel"
          mat-dialog-close
          class="ags-outline-btn ags-hmd44 btn-font16 ags-padding1624"
        >
          Cancel
        </button>
      </div>
      <div>
        <button
          [disabled]="panelForm.invalid"
          title="Submit"
          type="submit"
          class="ags-primary-btn ags-hmd44 btn-font16 ags-padding1624"
        >
          Schedule
        </button>
      </div>
    </div>
  </form>
</div>
