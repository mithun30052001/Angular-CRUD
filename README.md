https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


req.ts

import { Component, Inject, OnInit } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import { AuthService } from '@/src/app/shared/services/auth.service';
import {
  FormBuilder,
  FormGroup,
  Validators,
  FormControl,
} from '@angular/forms';
import { Job } from '@/src/app/shared/services/job-posting.service';
import { UserProfile } from '@/src/app/interfaces/app-interface';
import { Observable, of } from 'rxjs';
import { debounceTime, switchMap, catchError, startWith } from 'rxjs/operators';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent implements OnInit {
  reqForm: FormGroup;
  jobAutoComplete$!: Observable<Job[]>;
  spocAutoComplete$!: Observable<UserProfile[]>;
  managerAutoComplete$!: Observable<UserProfile[]>;

  // Form fields (jobId, spoc, manager will be auto-complete)
  formFields = [
    {
      name: 'jobId',
      label: 'Job',
      placeholder: 'Search for a job',
      type: 'autocomplete',  // Use 'autocomplete' for autocomplete fields
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
      placeholder: 'Search for SPOC',
      type: 'autocomplete',  // Use 'autocomplete' for autocomplete fields
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
      placeholder: 'Search for manager',
      type: 'autocomplete',  // Use 'autocomplete' for autocomplete fields
    },
  ];

  constructor(
    private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public rowData: any,
    private reqService: ReqService,
    private authService: AuthService,
    private fb: FormBuilder
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

  ngOnInit() {
    // Initialize autocomplete observables
    this.jobAutoComplete$ = this.reqService.getJobs().pipe(
      debounceTime(300),
      switchMap((term) => {
        return this.reqService.searchJobs(term).pipe(
          catchError(() => of([]))  // Handle errors by returning empty array
        );
      })
    );

    this.spocAutoComplete$ = this.authService.hrSearch('').pipe(
      debounceTime(300),
      switchMap((term) => {
        return this.authService.hrSearch(term).pipe(
          catchError(() => of([]))  // Handle errors by returning empty array
        );
      })
    );

    this.managerAutoComplete$ = this.authService.hrSearch('').pipe(
      debounceTime(300),
      switchMap((term) => {
        return this.authService.hrSearch(term).pipe(
          catchError(() => of([]))  // Handle errors by returning empty array
        );
      })
    );
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
  
  <div class="bodyinfo">
    <!-- Info section remains the same -->
    <div class="row">
      <div class="col-sm-3">
        <div class="info-inner">
          <h5>Request Type</h5>
          <p>{{ rowData.requestType }}</p>
        </div>
      </div>
      <!-- Additional Info Fields Here -->
    </div>
  </div>

  <form [formGroup]="reqForm" (ngSubmit)="onSubmit()">
    <div class="req-edit-body">
      <div class="row">
        <div class="col-lg-4 col-sm-4" *ngFor="let field of formFields">
          <div class="form-group ags-form-group">
            <label for="{{ field.name }}" class="form-label">
              {{ field.label }}<span class="required"></span>
            </label>

            <mat-form-field *ngIf="field.type === 'text'">
              <input
                matInput
                [formControlName]="field.name"
                [placeholder]="field.placeholder"
                required
              />
            </mat-form-field>

            <mat-form-field *ngIf="field.type === 'autocomplete'">
              <input
                matInput
                [formControlName]="field.name"
                [placeholder]="field.placeholder"
                [matAutocomplete]="auto"
                required
              />
              <mat-autocomplete #auto="matAutocomplete">
                <mat-option *ngFor="let item of getAutocompleteOptions(field.name) | async" [value]="item">
                  {{ item.name || item.jobTitle }}
                </mat-option>
              </mat-autocomplete>
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
