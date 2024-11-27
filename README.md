https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


req.ts

import { Component, Inject, OnInit } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import { FormBuilder, FormGroup, Validators, FormControl } from '@angular/forms';
import { Requisition } from '../req-mapping/req-mapping.component';
import { Job } from '@/src/app/shared/services/job-posting.service';
import { ElasticSearchResponse } from '@/src/app/interfaces/app-interface';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import { AuthService } from '@/src/app/shared/services/auth.service';
import { Observable, of } from 'rxjs';
import { catchError, debounceTime, map, switchMap, startWith } from 'rxjs/operators';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent implements OnInit {
  reqForm: FormGroup;
  jobs: Job[] = [];
  hrUsers: any[] = [];
  
  // Autocomplete fields
  jobAutoCompleteControl = new FormControl();
  spocAutoCompleteControl = new FormControl();
  managerAutoCompleteControl = new FormControl();

  constructor(
    private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public data: Requisition,
    private reqService: ReqService,
    private fb: FormBuilder,
    private applicationService: ApplicationService,
    private authService: AuthService
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
  }

  ngOnInit(): void {
    this.getJobs();
    this.getHRUsers();

    // Set initial values if data exists
    if (this.data) {
      this.reqForm.patchValue({
        jobId: this.data.jobId,
        requisitionId: this.data.requisitionId,
        spoc: this.data.spoc,
        manager: this.data.manager,
        closureDate: this.data.closureDate,
        onboardingDate: this.data.onboardingDate,
        published: this.data.published,
      });
    }

    this.jobAutoCompleteControl.valueChanges.pipe(
      debounceTime(300),
      switchMap((value) => this.lookupJob(value))
    ).subscribe();

    this.spocAutoCompleteControl.valueChanges.pipe(
      debounceTime(300),
      switchMap((value) => this.lookupHR(value))
    ).subscribe();

    this.managerAutoCompleteControl.valueChanges.pipe(
      debounceTime(300),
      switchMap((value) => this.lookupHR(value))
    ).subscribe();
  }

  // Fetch job listings for autocomplete
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

  // HR User lookup for spoc and manager autocomplete
  getHRUsers() {
    this.authService.hrSearch('').subscribe((users) => {
      this.hrUsers = users;
    });
  }

  // Transform list response
  transformList(response: ElasticSearchResponse<any>) {
    return response.hits.hits.map((job) => {
      return job._source;
    });
  }

  // Job lookup for autocomplete
  lookupJob(value: string): Observable<any> {
    if (value) {
      return this.applicationService.searchJobs(value).pipe(
        map((results) => results),
        catchError(() => of([]))
      );
    }
    return of([]);
  }

  // HR lookup for autocomplete
  lookupHR(value: string): Observable<any> {
    if (value) {
      return this.authService.hrSearch(value).pipe(
        map((results) => results),
        catchError(() => of([]))
      );
    }
    return of([]);
  }

  closeModal() {
    this.dialogRef.close();
  }

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


req.html

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
      <div class="row">
        <div class="col-lg-6 col-sm-6" *ngFor="let field of formFields">
          <div class="form-group ags-form-group" *ngIf="field.name !== 'spoc' && field.name !== 'manager' && field.name !== 'jobId'">
            <label for="{{field.name}}" class="form-label">{{field.label}}<span class="required"></span></label>
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

          <!-- Job Autocomplete -->
          <div class="form-group ags-form-group" *ngIf="field.name === 'jobId'">
            <label for="jobId" class="form-label">Job<span class="required"></span></label>
            <mat-form-field>
              <input
                matInput
                [formControl]="jobAutoCompleteControl"
                placeholder="Search Job"
                [matAutocomplete]="autoJob"
                required
              />
              <mat-autocomplete #autoJob="matAutocomplete">
                <mat-option *ngFor="let job of jobs" [value]="job">
                  {{ job.jobTitle }} - {{ job.location }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>

          <!-- SPOC Autocomplete -->
          <div class="form-group ags-form-group" *ngIf="field.name === 'spoc'">
            <label for="spoc" class="form-label">SPOC<span class="required"></span></label>
            <mat-form-field>
              <input
                matInput
                [formControl]="spocAutoCompleteControl"
                placeholder="Search SPOC"
                [matAutocomplete]="autoSpoc"
                required
              />
              <mat-autocomplete #autoSpoc="matAutocomplete">
                <mat-option *ngFor="let user of hrUsers" [value]="user">
                  {{ user.fullName || user.email }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>

          <!-- Manager Autocomplete -->
          <div class="form-group ags-form-group" *ngIf="field.name === 'manager'">
            <label for="manager" class="form-label">Manager<span class="required"></span></label>
            <mat-form-field>
              <input
                matInput
                [formControl]="managerAutoCompleteControl"
                placeholder="Search Manager"
                [matAutocomplete]="autoManager"
                required
              />
              <mat-autocomplete #autoManager="matAutocomplete">
                <mat-option *ngFor="let user of hrUsers" [value]="user">
                  {{ user.fullName || user.email }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>

        <div class="col-lg-6 col-sm-6">
          <div class="form-group ags-form-group">
            <label for="published" class="form-label">Publish<span class="required"></span></label>
            <mat-radio-group formControlName="published">
              <mat-radio-button value="true">Yes</mat-radio-button>
              <mat-radio-button value="false">No</mat-radio-button>
            </mat-radio-group>
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
          Save
        </button>
      </div>
    </div>
  </form>
</div>
