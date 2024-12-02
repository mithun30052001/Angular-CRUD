https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

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
import { ReqService } from '@/src/app/shared/services/req.service';
import { JobService } from '@/src/app/shared/services/job-posting.service';

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
  public jobDetails: any[] = [];

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
    private reqService: ReqService,
    private jobService: JobService,
    private dialog: MatDialog,
    public dialogRef: MatDialogRef<PanelScheduleDialogComponent>
  ) {
    super(media);
    this.isAvailableTimeChecked = false;
    if (data.job?.jobId) this.jobId = data.job?.jobId;
    if (this.jobId) {
      var reqIdList: any = [];
      this.reqService.getPublishedRequisitions(
        {
          jobId :this.jobId
        }
      ).subscribe((res) => {
        res.forEach((x: { requisitionId: any }) => {
          reqIdList.push(x);
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
    console.log('EVENTDATA INSIDE CONSTRUCTOR', this.eventData);
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
    console.log('SELECTED PANEL', event.option.value);
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
     this.reqService.getPublishedRequisitions(
        {
          jobId :this.jobId
        }
      ).subscribe((res) => {
        res.forEach((x: { requisitionId: any }) => {
          reqIdList.push(x);
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
      .getCandidatesForPanel(value?.toLowerCase(), uniq(toAvoid), this.jobId, this.panelForm.get('requisitionId')?.toString())
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
    console.log('eventData inside dialog', this.eventData.panelMembers);
    if (this.eventData?.id) {
      this.isEditMode = true;
      this.jobAcControl.disable();
      this.dialogHeading = 'Edit Panel Schedule';
      this.scheduleButtonText = 'Reschedule';
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

  async populateFormWithEventData(eventData: any): Promise<void> {
    console.log('EVENT DATA INSIDE POPULATEFORM', eventData);

    this.jobLookup(eventData.title).subscribe(
      (jobDetails: any) => {
        console.log('JOB DETAILS', jobDetails);
      },
      (error) => {
        console.error('ERROR FETCHING JOB DETAILS', error);
      }
    );

    const jobPostingRes: any = await this.jobService.getJobPostingById(eventData.meta.jobId);
    if (jobPostingRes) {
      this.jobId = jobPostingRes?.id;
    }
   
    this.panelForm.patchValue({
      jobId: {
        id: this.jobId,
        display: jobPostingRes?.jobTitle,
      },
    });

    const reqIdList: any = [];
    const requisitionsRes: any = await this.reqService.getPublishedRequisitions({ jobId: this.jobId });
    requisitionsRes.forEach((x: { requisitionId: any }) => reqIdList.push(x));
    this.reqAc$ = reqIdList;

    // this.panelForm.patchValue({
    //   jobId: {
    //     id: 'e5d2e945-6b57-42ae-90f8-37651b5996c2',
    //     display: 'Voice - Experience - HB - Chennai - Chennai - (Experienced)',
    //   },
    // });

   

    if (eventData?.requisionId != null) {
        this.panelForm.patchValue({
        requisitonId: eventData.requisitionId,
     });
    }

    if (eventData.meta.candidates && eventData.meta.candidates.length > 0) {
      console.log('EVENT CANDIDATES', eventData.meta.candidates);
      //this.selectedCandidates = eventData.candidates.filter((candidate: any) => candidate.status == null || candidate.status == undefined);
      this.selectedCandidates = eventData.meta.candidates;
      this.applicationAcControl.setValue(
        this.selectedCandidates.map((candidate: any) => candidate.applicationId)
      );
    }

    // Set Primary Panel Member
    const primaryPanelMember = eventData.meta.panelMembers.find(
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
    const secondaryPanelMembers = eventData.meta.panelMembers.filter(
      (pm: any) => pm.primaryPanelMember === false
    );
    console.log('SECONDARY PANEL MEMBERS', secondaryPanelMembers);

    this.secondaryPanelMembers = secondaryPanelMembers.map((sec: any) => {
      return {
        id: sec.employeeId,
        employeeId: sec.employeeId,
        employeeName: sec.name,
        emailId: sec.email,
      };
    });

    this.selectedPanel = this.secondaryPanelMembers;

    const secondaryPanelMemberIds = secondaryPanelMembers.map(
      (sec: any) => sec.employeeId
    );

    console.log('Secondary Ids', secondaryPanelMemberIds);

    this.panelForm.patchValue({
      secondaryEmployeeIds: secondaryPanelMemberIds,
    });

    // Set Date and Time
    const start = new Date(eventData.start);
    this.panelForm.patchValue({
      date: start,
    });

    // const currentTimeFormat:any = {
    //   currentTimeFormat:
    //     '+05:30 India Time - Mumbai, Delhi, Bengaluru, Hyderābād',
    // };
    // console.log('MY CITIES', this.displayFn(currentTimeFormat));

    // Set Timezone
    console.log('EVENT TIMEZONE', eventData.meta.timezone);
    
    this.panelForm.patchValue({
      timezone: {
        name: eventData.meta.timezone,
        mainCities: ['Mumbai', 'Delhi', 'Bengaluru', 'Hyderābād'],
        currentTimeFormat:
          '+05:30 India Time - Mumbai, Delhi, Bengaluru, Hyderābād',
      },
    });
    this.checkCal();

    const startDate = new Date(eventData.start);
    const endDate = new Date(eventData.end);

    // Function to format the date in "yyyy-MM-ddTHH:mm:ss" format
    const formatDate = (date: Date): string => {
      const year = date.getFullYear();
      const month = (date.getMonth() + 1).toString().padStart(2, '0');
      const day = date.getDate().toString().padStart(2, '0');
      const hours = date.getHours().toString().padStart(2, '0');
      const minutes = date.getMinutes().toString().padStart(2, '0');
      const seconds = date.getSeconds().toString().padStart(2, '0');

      return `${year}-${month}-${day}T${hours}:${minutes}:${seconds}`;
    };

    // Format the start and end time
    const formattedStartTime = formatDate(startDate);
    const formattedEndTime = formatDate(endDate);

   // Now patch the form with the formatted values
   
    this.panelForm.patchValue({
      timings: {
        startTime : formattedStartTime,
        endTime : formattedEndTime,
      },
    });
  

    // this.panelForm.patchValue({
    //   timings: {
    //     startTime: new Date(eventData.start),
    //     endTime: new Date(eventData.end),
    //   },
    // });

    console.log('MY UPDATION FORM', this.panelForm.value);
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
        console.log('JOB TITLE IN ANOTHER METHOD', this.data?.job?.jobTitle);
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
    console.log('Form value', this.panelForm?.value);

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

     if (this.isEditMode) {
          payload.type = 'WHOLE_PANEL_UPDATE';  // Append 'type' if it's in edit mode
    }

    console.log('Our Payload', payload.secondaryEmployeeIds);

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

panel-schedule.component.html

<div class="upload-candidate-modal">
  <div class="upload-candidate-header">
    <h5 class="heading-text">
      {{ dialogHeading }}
    </h5>
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
                placeholder="select requisition number"
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
                  [value]="item.requisitionId"
                >
                   {{ item.requisitionId }} ({{ item.clientName }})
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
          {{scheduleButtonText}}
        </button>
      </div>
    </div>
  </form>
</div>




needed value format in populateformwitheventdata function
timezone: {
        name: eventData.meta.timezone,
        mainCities: ['Mumbai', 'Delhi', 'Bengaluru', 'Hyderābād'],
        currentTimeFormat:
          '+05:30 India Time - Mumbai, Delhi, Bengaluru, Hyderābād',
  },

selected option value
{
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

