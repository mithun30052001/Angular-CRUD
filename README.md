https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


req-mapping.ts

import { Component, Inject } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import { UserProfile } from '@/src/app/interfaces/app-interface';
import {
  FormBuilder,
  FormGroup,
  Validators,
  FormControl,
  AbstractControl,
  ValidationErrors,
} from '@angular/forms';
import { Requisition } from '../req-mapping/req-mapping.component';
import { Job } from '@/src/app/shared/services/job-posting.service';
import { ElasticSearchResponse } from '@/src/app/interfaces/app-interface';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import {
  catchError,
  debounceTime,
  map,
  Observable,
  of,
  startWith,
  switchMap,
} from 'rxjs';
import { AuthService } from '@/src/app/shared/services/auth.service';
import { Router } from '@angular/router';


@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;
  jobs: Job[] = [];
  public managerComplete$: any;
  public spocComplete$: any;
  public managerControl = new FormControl<any>({ value: '', disabled: false }, [
    Validators.required,
    this.validateHrIdCtrl,
  ]);
  public spocControl = new FormControl<any>({ value: '', disabled: false }, [
    Validators.required,
    this.validateHrIdCtrl,
  ]);
  public jobControl = new FormControl<any>({ value: '' }, Validators.required);

  form: FormGroup = new FormGroup({
    reqForm: new FormGroup({
      jobId: this.jobControl,
      requisitionId: new FormControl(''),
      spoc: this.spocControl,
      closureDate: new FormControl(''),
      onboardingDate: new FormControl(''),
      manager: this.managerControl,
    }),
  });

  formFields = [
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
  ];


  originalOpenNumbers: number | null = null;
  openNumbersChanged = false;
  isHiringUpdate: boolean;

  constructor(
    private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public rowData: any,
    private reqService: ReqService,
    private fb: FormBuilder,
    public router: Router,
    private applicationService: ApplicationService,
    private authService: AuthService
  ) {
    const closureDateObject = new Date(rowData?.closureDate);
    const formattedClosureDate = closureDateObject.toISOString().split('T')[0];
    this.isHiringUpdate = this.router?.url?.includes('h/hiring-update');
      const onboardingDateObject = new Date(rowData?.onboardingDate);
      const formattedOnboardingDate = onboardingDateObject
        ?.toISOString()
        .split('T')[0];

    this.reqForm = this.fb.group({
      jobId: rowData?.job?.id ?? this.jobControl,
      requisitionId: [rowData?.requisitionId ?? '', Validators.required],
      spoc: rowData?.hrSpoc ?? this.spocControl,
      closureDate: [formattedClosureDate ?? '', Validators.required],
      onboardingDate: [formattedOnboardingDate ?? '', Validators.required],
      manager: this.managerControl,
      published: [true, Validators.required],
    });
    console.log('DIALOG DATA', rowData);
    console.log("REQFORM", rowData?.hrSpoc);
    if (this.isHiringUpdate === true) {
      this.originalOpenNumbers = rowData.openNumbers;
      this.reqForm.addControl(
        'openNumbers',
        new FormControl(rowData.openNumbers, Validators.required)
      );
      this.reqForm.addControl('openNumbersProof', new FormControl(null));

      this.reqForm.get('openNumbers')?.valueChanges.subscribe((newValue) => {
        this.openNumbersChanged = newValue !== this.originalOpenNumbers;
      });
    } else {
      this.formFields.unshift({
        name: 'requisitionId',
        label: 'Requisition ID',
        placeholder: 'Enter requisition ID',
        type: 'text',
      });
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

  ngOnInit(): void {
    console.log('MY DIALOG DATA INIT', this.rowData);
    this.isHiringUpdate = this.router?.url?.includes('h/hiring-update');
    this.getJobs();

    this.managerComplete$ = this.managerControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      switchMap((value) => {
        if (value !== '') {
          if (value?.fullName) {
            return of(null);
          }
          return this.lookup(value);
        } else {
          return this.lookup('');
        }
      })
    );

    this.spocComplete$ = this.spocControl.valueChanges.pipe(
      startWith(''),
      debounceTime(300),
      switchMap((value) => {
        if (value !== '') {
          if (value?.fullName) {
            return of(null);
          }
          return this.lookup(value);
        } else {
          return this.lookup('');
        }
      })
    );
  }

  lookup(value: string): Observable<any> {
    return this.authService.hrSearch(value?.toLowerCase()).pipe(
      map((results) => results),
      catchError((_) => {
        return of(null);
      })
    );
  }

  displayFullNameFn(option: UserProfile | null) {
    console.log('Option', option);
    return option ? option?.fullName ?? option?.email?.split('@')[0] : '';
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
        console.log('MY JOBS', this.jobs);
      });
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.reqForm.valid) {
      const formValues = this.reqForm.value;
      console.log('Manager', formValues.manager);
      formValues.manager = formValues.manager.id;
      formValues.spoc = formValues.spoc.id;
      console.log(formValues);
      // if (this.isHiringUpdate === true) {
      //   console.log('OpenNumber', formValues.openNumbers);
      //   const proofFile = this.reqForm.get('openNumbersProof')?.value;
      //   console.log('Proof file', proofFile);
      //   const requestData = {
      //     jobRequisition: formValues,
      //     file: proofFile,
      //   };

      //   this.reqService.updatePublishedRequisitions(requestData).subscribe(
      //     (response) => {
      //       console.log(
      //         'Published requisition updated successfully:',
      //         response
      //       );
      //       this.dialogRef.close(true);
      //     },
      //     (error) => {
      //       console.error('Error updating published requisition:', error);
      //     }
      //   );
      // } else {
      //   this.reqService.updateJobRequisition(formValues).subscribe(
      //     (response) => {
      //       console.log('Job requisition updated successfully:', response);
      //       this.dialogRef.close(true);
      //     },
      //     (error) => {
      //       console.error('Error updating job requisition:', error);
      //     }
      //   );
      // }
    }
  }
}

req-mapping-edit-dialog.html

<div class="req-edit-modal">
  <div class="req-edit-header">
    <h5 class="heading-text">Req Mapping Edit</h5>
    <div>
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>
    </div>
  </div>
  <form [formGroup]="reqForm" (ngSubmit)="onSubmit()">
    <div class="req-edit-body">
      <div class="bodyinfo">
        <div class="row">
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Requisition ID</h5>
              <p>{{ rowData.requisitionId }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Process</h5>
              <p>{{ rowData.process }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Client</h5>
              <p>{{ rowData.clientName }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Speciality</h5>
              <p>{{ rowData.speciality }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Job Type</h5>
              <p>{{ rowData.jobType }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Location</h5>
              <p>{{ rowData.location }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Shift</h5>
              <p>{{ rowData.shift.split('_')[1] }}</p>
            </div>
          </div>
          <div class="col-sm-4">
            <div class="info-inner">
              <h5>Open Numbers</h5>
              <p>{{ rowData.openNumbers }}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-sm-4">
          <div class="form-group form-inner">
            <label class="form-label" for="manager"
              >Manager<span class="required"></span
            ></label>
            <mat-form-field class="example-form-field">
              <input
                [formControl]="managerControl"
                type="text"
                placeholder="search manager"
                aria-label="string"
                matInput
                [matAutocomplete]="managerAuto"
              />
              <mat-autocomplete
                [displayWith]="displayFullNameFn"
                #managerAuto="matAutocomplete"
              >
                <mat-option
                  *ngFor="
                    let item of managerComplete$ | async;
                    let index = index
                  "
                  [value]="item"
                >
                  {{ item.fullName ?? item.email.split('@')[0] | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="form-group form-inner">
            <label class="form-label" for="spoc"
              >SPOC<span class="required"></span
            ></label>
            <mat-form-field class="example-form-field">
              <input
                [formControl]="spocControl"
                type="text"
                placeholder="Search SPOC"
                aria-label="string"
                matInput
                [matAutocomplete]="spocAuto"
              />
              <mat-autocomplete
                [displayWith]="displayFullNameFn"
                #spocAuto="matAutocomplete"
              >
                <mat-option
                  *ngFor="let item of spocComplete$ | async; let index = index"
                  [value]="item"
                >
                  {{ item.fullName ?? item.email.split('@')[0] | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-sm-4">
          <div class="form-group form-inner">
            <label class="form-label" for="job"
              >Job<span class="required"></span
            ></label>
            <mat-form-field class="example-form-field">
              <mat-select name="job" multiple="false" formControlName="jobId">
                <mat-option *ngFor="let job of jobs" [value]="job.id">
                  {{ job.jobTitle | textFill }} -
                  {{ job.location | textFill | startCase }}
                  ({{ job.experienceLevel | textFill }})
                </mat-option>
              </mat-select>
            </mat-form-field>
          </div>
        </div>
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

        <div class="col-lg-4 col-sm-4" *ngIf="isHiringUpdate === true">
        <div class="form-group ags-form-group">
          <label for="openNumbers" class="form-label">
            Open Numbers<span class="required"></span>
          </label>
          <mat-form-field>
            <input
              matInput
              formControlName="openNumbers"
              type="number"
              placeholder="Enter open numbers"
              required
            />
          </mat-form-field>
        </div>
      </div>

       <div class="col-lg-4 col-sm-4" *ngIf="isHiringUpdate === true && openNumbersChanged">
        <div class="form-group ags-form-group">
          <label for="openNumbersProof" class="form-label">
            Upload Proof<span class="required"></span>
          </label>
          <input
            type="file"
            class="form-control"
            formControlName="openNumbersProof"
            accept=".pdf,.png,.jpg,.jpeg"
          />
        </div>
      </div>

        <!-- <div class="col-lg-6 col-sm-6">
          <div class="form-group ags-form-group">
            <label for="published" class="form-label"
              >Publish<span class="required"></span
            ></label>
            <mat-radio-group formControlName="published">
              <mat-radio-button value="true">Yes</mat-radio-button>
              <mat-radio-button value="false">No</mat-radio-button>
            </mat-radio-group>
          </div>
        </div> -->
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

