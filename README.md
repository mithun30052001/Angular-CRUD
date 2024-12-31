https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

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
  public reqAcControl: FormControl<any>;
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


  secSelected(event: MatAutocompleteSelectedEvent): void {
    this.selectedPanel.push(event.option.value);
    console.log('SELECTED PANEL', event.option.value);
    this.secAutoCompleteInput.nativeElement.value = '';
    this.secondaryPanelAcControl.setValue(null);
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
    console.log("SELECTED TIMEZONE", event.option.value);
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

  }
  ngOnInit(): void {
    if (this.eventData?.id) {
      this.reqAcControl = new FormControl<any>({ value: '', disabled: true }, [
        Validators.required,
      ]);
    }
    else {
       this.reqAcControl = new FormControl<any>({ value: '', disabled: false }, [
         Validators.required,
       ]);
    }
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
