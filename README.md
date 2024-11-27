https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

hr-edit.component.ts

import { CandidateQuickEditComponent } from '@/src/app/candidate/components/candidate-profile-menu/candidate-quick-edit/candidate-quick-edit.component';
import { MessageModalsComponent } from '@/src/app/common-modules/message-modals/message-modals.component';
import {
  ApplicationStatus,
  CandidateApplication,
  ElasticSearchResponse,
  UserProfile,
} from '@/src/app/interfaces/app-interface';
import { ResponsiveViewComponent } from '@/src/app/shared/components/responsive-view/responsive-view.component';
import { AuthService } from '@/src/app/shared/services/auth.service';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import { Job } from '@/src/app/shared/services/job-posting.service';
import { UtilService } from '@/src/app/shared/services/util.service';
import { MediaMatcher } from '@angular/cdk/layout';
import { Component, Inject, OnInit } from '@angular/core';
import {
  AbstractControl,
  FormBuilder,
  FormControl,
  FormGroup,
  ValidationErrors,
  Validators,
} from '@angular/forms';
import {
  MAT_DIALOG_DATA,
  MatDialog,
  MatDialogRef,
} from '@angular/material/dialog';
import {
  catchError,
  debounceTime,
  map,
  Observable,
  of,
  startWith,
  switchMap,
} from 'rxjs';

@Component({
  selector: 'app-hr-quick-edit',
  templateUrl: './hr-quick-edit.component.html',
  styleUrls: ['./hr-quick-edit.component.scss'],
})
export class HrQuickEditComponent
  extends ResponsiveViewComponent
  implements OnInit
{
  form: FormGroup = new FormGroup({
    hr: new FormControl(''),
    jobId: new FormControl(''),
  });
  jobs: Job[] = [];
  hrDetailList: any;
  userId = '';
  get f(): { [key: string]: AbstractControl } {
    return this.form.controls;
  }

  isAdmin = false;

  public hrAutoComplete$: any;
  public autoCompleteControl = new FormControl<any>(
    { value: '', disabled: false },
    [Validators.required, this.validateHrIdCtrl]
  );

  constructor(
    private formBuilder: FormBuilder,
    private authService: AuthService,
    private dialogRef: MatDialogRef<CandidateQuickEditComponent>,
    media: MediaMatcher,
    public dialog: MatDialog,
    @Inject(MAT_DIALOG_DATA)
    public data: CandidateApplication,
    private utilService: UtilService,
    private applicationService: ApplicationService
  ) {
    super(media);
  }

  lookup(value: string): Observable<any> {
    return this.authService.hrSearch(value?.toLowerCase()).pipe(
      // map the item property of the results as our return object
      map((results) => results),
      // catch errors
      catchError((_) => {
        return of(null);
      })
    );
  }

  displayFullNameFn(option: UserProfile | null) {
    return option ? option.fullName ?? option.email.split('@')[0] : '';
  }

  isJobEditable = (status: ApplicationStatus) => {
    return [
      'NONE',
      'REGISTERED',
      'CREATED',
      'WALKEDIN',
      'WAITING_FOR_HRROUND',
    ].includes(status);
  };

  initiateForm() {
    this.isAdmin = this.utilService.checkAdminAccess();
    this.userId = localStorage.getItem('id') as string;
    if (this.data) {
      this.form = this.formBuilder.group({
        hr: this.autoCompleteControl,
        jobId: [
          { value: '', disabled: !this.isJobEditable(this.data.status) },
          Validators.required,
        ],
      });

      if (this.data?.hr?.id) {
        const hrField = this.form.get('hr');
        hrField?.patchValue(this.data.hr);
        console.log(hrField);
        if (!this.isAdmin) hrField?.disable();
      }
      if (this.data?.job?.id) {
        const jobIdField = this.form.get('jobId');
        jobIdField?.patchValue(this.data.job.id);
      }
    }
  }

  validateHrIdCtrl(ctrl: AbstractControl): ValidationErrors | null {
    const val = ctrl.value;
    if (!val || val === '' || !val?.id) {
      return {
        error: true,
      };
    }
    return null;
  }

  ngOnInit() {
    this.hrAutoComplete$ = this.autoCompleteControl.valueChanges.pipe(
      startWith(''),
      // delay emits
      debounceTime(300),
      // use switch map so as to cancel previous subscribed events, before creating new once
      switchMap((value) => {
        if (value !== '') {
          if (value?.fullName) {
            return of(null);
          }
          // lookup from github
          return this.lookup(value);
        } else {
          // if no value is present, return null
          return this.lookup('');
        }
      })
    );
    this.initiateForm();

    this.getJobs();
  }

  transformList(response: ElasticSearchResponse<any>) {
    return response.hits.hits.map((job) => {
      return job._source;
    });
  }

  getJobs() {
    this.applicationService
      .getAllJobs({
        orderBy: 'desc',
        limit: 500,
        offset: 0,
      })
      .then((res) => {
        this.jobs = this.transformList(res);
      });
  }

  submitForm() {
    const payload = {
      ...this.form.value,
      ...(this.form.value?.hr?.id && { hr: this.form.value.hr.id }),
      referralUpdate: this.data.status === 'NONE',
    };

    this.applicationService
      .quickUpdateCandidateApplicationByHr(this.data.id, payload)
      .then((_res) => {
        this.dialog
          .open(MessageModalsComponent, {
            maxWidth: this.mobileView ? '500px' : '100%',
            width: this.mobileView ? '100%' : '30%',
            ...(this.mobileView && {
              height: 'auto',
              position: { bottom: '0' },
            }),
            closeOnNavigation: true,
            disableClose: false,
            data: this.mobileView
              ? 'hrQuickEditSaveSuccessfullyMobile'
              : 'hrQuickEditSaveSuccessfullyWeb',
          })
          .afterClosed()
          .subscribe(() => {
            this.dialogRef.close(1);
          });
      });
  }

  closeModal() {
    this.dialogRef.close();
  }
}



hr-edit.component.html

<!-- Mobile View -->

<div class="quick-edit-modal">
  <form [formGroup]="form" novalidate>
    <div class="heading">
      <div class="heading-text">Job Management</div>
      <div>
        <app-icon icon="close" (click)="closeModal()"></app-icon>
      </div>
    </div>
    <div class="quick-edit-body">
      <div class="row">
        <div class="col-sm-12">
          <div class="form-group form-inner">
            <label class="form-label" for="hr"
              >Tag HR<span class="required"></span
            ></label>
            <mat-form-field
              [matTooltip]="
                f['hr'].disabled ? 'Only admin users can update this field' : ''
              "
              class="example-form-field"
            >
              <!-- <mat-select name="hr" multiple="false" formControlName="hr">
                <mat-option
                  *ngFor="let option of hrDetailList"
                  [value]="option.id"
                >
                  {{ option.email }}
                </mat-option>
              </mat-select> -->
              <input
                [formControl]="autoCompleteControl"
                type="text"
                placeholder="search HR"
                aria-label="string"
                matInput
                [matAutocomplete]="auto"
              />
              <mat-autocomplete
                [displayWith]="displayFullNameFn"
                autoActiveFirstOption
                #auto="matAutocomplete"
              >
                <mat-option
                  *ngFor="
                    let item of hrAutoComplete$ | async;
                    let index = index
                  "
                  [value]="item"
                >
                  {{ item.fullName ?? item.email.split('@')[0] | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
            <div
              *ngIf="f['hr'].touched && f['hr'].dirty && f['hr'].invalid"
              class="form-invalid"
            >
              <p>HR is mandatory</p>
            </div>
          </div>
        </div>
      </div>

      <!-- Status -->
      <div class="row">
        <div class="col-sm-12">
          <div class="form-group form-inner">
            <label class="form-label" for="job"
              >Job<span class="required"></span
            ></label>
            <mat-form-field
              [matTooltip]="
                f['jobId'].disabled ? 'Cannot change job right now' : ''
              "
              class="example-form-field"
            >
              <mat-select name="job" multiple="false" formControlName="jobId">
                <mat-option *ngFor="let job of jobs" [value]="job.id">
                  {{ job.jobTitle | textFill }} -
                  {{ job.location | textFill | startCase }}
                  ({{ job.experienceLevel | textFill }})
                </mat-option>
              </mat-select>

              <div
                *ngIf="
                  f['jobId'].touched && f['jobId'].dirty && f['jobId'].invalid
                "
                class="form-invalid"
              >
                <p>Job is mandatory</p>
              </div>
            </mat-form-field>
          </div>
        </div>
      </div>

      <!-- Remarks -->
      <!-- <div class="row">
        <div class="col-sm-12">
          <div class="form-inner">
            <label class="form-label" for="remarks">Remarks</label>
            <div>
              <input
                name="remarks"
                matInput
                formControlName="remark"
                class="form-control"
                placeholder="Remarks"
              />
              <div
                *ngIf="
                  f['remark'].touched &&
                  f['remark'].dirty &&
                  f['remark'].invalid
                "
                class="form-invalid"
              >
                <p>Cannot exceed 200 characters</p>
              </div>
            </div>
          </div>
        </div>
      </div> -->
    </div>
    <div class="quick-edit-footer">
      <div class="row justify-content-end">
        <div class="col-sm-2">
          <button
            title="Save"
            class="ags-primary-btn ags-hlg48 ags-padding1216 btn-font16"
            (click)="submitForm()"
            [disabled]="
              form.invalid || (f['hr'].disabled && f['jobId'].disabled)
            "
          >
            Save
          </button>
        </div>
      </div>
    </div>
  </form>
</div>
















req-mapping-edit-dialog.component.ts

import { Component, Inject } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import {
  FormBuilder,
  FormGroup,
  Validators,
  FormControl,
} from '@angular/forms';
import { Requisition } from '../req-mapping/req-mapping.component';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;
  form: FormGroup = new FormGroup({
    reqForm: new FormGroup({
      jobId: new FormControl(''),
      requisitionId: new FormControl(''),
      spoc: new FormControl(''),
      closureDate: new FormControl(''),
      onboardingDate: new FormControl(''),
      manager: new FormControl(''),
      // country: new FormControl(''),
      // pinCode: new FormControl(0),
      // code: new FormControl<number>(0),
    }),
  });
  formFields = [
    {
      name: 'jobId',
      label: 'Job',
      placeholder: 'Enter job ID',
      type: 'text',
    },
    {
      name: 'requisitionId',
      label: 'Requisition ID',
      placeholder: 'Enter requisition ID',
      type: 'text',
    },
    {
      name: 'spoc',
      label: 'SPOC',
      placeholder: 'Enter SPOC name',
      type: 'text',
    },
    {
      name: 'closureDate',
      label: 'Closure Date',
      placeholder: 'Enter closure date',
      type: 'date',
    },
    {
      name: 'onboardingDate',
      label: 'Onboarding Date',
      placeholder: 'Enter onboarding date',
      type: 'date',
    },
    {
      name: 'manager',
      label: 'Manager',
      placeholder: 'Enter manager name',
      type: 'text',
    },
  ];

  constructor(
    private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public rowData: Requisition,
    private reqService: ReqService,
    private fb: FormBuilder,
    private formBuilder: FormBuilder
  ) {
    this.reqForm = this.fb.group({
      jobId: ['', Validators.required],
      requisitionId: ['', Validators.required],
      spoc: ['', Validators.required],
      closureDate: ['', Validators.required],
      onboardingDate: ['', Validators.required],
      manager: ['', Validators.required],
      published: [true, Validators.required],
    });
    console.log('DIALOG DATA', rowData);
  }

  closeModal() {
    this.dialogRef.close();
  }

  //  reqForm: this.formBuilder.group(
  //         this.addressFormControls()
  //       ),
  onSubmit() {
    if (this.reqForm.valid) {
      const formValues = this.reqForm.value;
      formValues.published = formValues.published === 'true' ? true : false;
      this.reqService.updateJobRequisition(formValues).subscribe(
        (response) => {
          console.log('Job requisition updated successfully:', response);
          this.dialogRef.close(true);
        },
        (error) => {
          console.error('Error updating job requisition:', error);
        }
      );
    }
  }
}


req-mapping-edit-dialog.component.html


<div class="req-edit-modal">
  <div class="req-edit-header">
    <h5 class="heading-text">Req Mapping Edit</h5>
    <div>
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>
    </div>
  </div>
  <div class="bodyinfo">
    <div class="row">
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Request Type</h5>
          <p>{{ rowData.requestType }}</p>
        </div>
      </div>

      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Requisition ID</h5>
          <p>{{ rowData.requisitionId }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Process</h5>
          <p>{{ rowData.process }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Client</h5>
          <p>{{ rowData.clientName }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Speciality</h5>
          <p>{{ rowData.speciality }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Job Type</h5>
          <p>{{ rowData.jobType }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Location</h5>
          <p>{{ rowData.location }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Shift</h5>
          <p>{{ rowData.shift }}</p>
        </div>
      </div>
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Open Numbers</h5>
          <p>{{ rowData.openNumbers }}</p>
        </div>
      </div>
    </div>
  </div>
  <form [formGroup]="reqForm" (ngSubmit)="onSubmit()">
    <div class="req-edit-body">
      <div class="row">
        <div class="col-lg-4 col-sm-4" *ngFor="let field of formFields">
          <div class="form-group ags-form-group">
            <label for="{{ field.name }}" class="form-label"
              >{{ field.label }}<span class="required"></span
            ></label>
            <mat-form-field>
              <input
                matInput
                [formControlName]="field.name"
                [placeholder]="field.placeholder"
                [type]="field.type"
                required
              />
            </mat-form-field>
          </div>
        </div>
      </div>
    </div>

    <div class="req-edit-footer">
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
          title="Submit"
          type="submit"
          class="ags-primary-btn ags-hmd44 btn-font16 ags-padding1624"
          [disabled]="reqForm.invalid"
        >
          Publish
        </button>
      </div>
    </div>
  </form>
</div>


