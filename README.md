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


  timeZoneChange(event: MatAutocompleteSelectedEvent): void {
    this.min = new Date(
      new Date().toLocaleString('en', { timeZone: event.option.value.name }) //if timezone is changed - show the dates accordingly
    );
    this.checkCal();
  }

  reqSelected(event: MatAutocompleteSelectedEvent): void {
    this.panelForm.get('requisitionId')?.setValue(event.option.value);
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

