https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-map-dialog.ts

import { Component } from '@angular/core';
import { MatDialogRef } from '@angular/material/dialog';
import { ReqService } from 'path-to-req-service';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;

  // Define form fields for dynamic generation (including the new manager and published fields)
  formFields = [
    { name: 'jobId', label: 'Job ID', placeholder: 'Enter job ID', type: 'text' },
    { name: 'reqId', label: 'Requisition ID', placeholder: 'Enter requisition ID', type: 'text' },
    { name: 'spoc', label: 'SPOC', placeholder: 'Enter SPOC name', type: 'text' },
    { name: 'closureDate', label: 'Closure Date', placeholder: 'Enter closure date', type: 'date' },
    { name: 'onboardingDate', label: 'Onboarding Date', placeholder: 'Enter onboarding date', type: 'date' },
    { name: 'openNumbers', label: 'Open Numbers', placeholder: 'Enter open numbers', type: 'number' },
    { name: 'manager', label: 'Manager', placeholder: 'Enter manager name', type: 'text' },  // New field
  ];

  constructor(
    private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
    private reqService: ReqService,
    private fb: FormBuilder
  ) {
    this.reqForm = this.fb.group({
      jobId: ['', Validators.required],
      reqId: ['', Validators.required],
      spoc: ['', Validators.required],
      closureDate: ['', Validators.required],
      onboardingDate: ['', Validators.required],
      openNumbers: ['', [Validators.required, Validators.min(1)]],
      manager: ['', Validators.required],  // New manager field with validation
      published: [true, Validators.required], // New published radio button field with default value true
    });
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.reqForm.valid) {
      const formValues = this.reqForm.value;
      this.reqService.updateJobRequisition(formValues).subscribe(
        (response) => {
          console.log('Job requisition updated successfully:', response);
          this.dialogRef.close(true);  // Close dialog on success
        },
        (error) => {
          console.error('Error updating job requisition:', error);
        }
      );
    }
  }
}


req-map-edit.html

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
        <!-- Loop through the dynamic form fields -->
        <div class="col-lg-6 col-sm-6" *ngFor="let field of formFields">
          <div class="form-group ags-form-group">
            <label for="{{field.name}}" class="form-label">{{field.label}}<span class="required">*</span></label>
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

        <!-- Manager field (text input) -->
        <div class="col-lg-6 col-sm-6">
          <div class="form-group ags-form-group">
            <label for="manager" class="form-label">Manager <span class="required">*</span></label>
            <mat-form-field>
              <input
                matInput
                formControlName="manager"
                placeholder="Enter manager name"
                required
              />
            </mat-form-field>
          </div>
        </div>

        <!-- Published Radio Button -->
        <div class="col-lg-6 col-sm-6">
          <div class="form-group ags-form-group">
            <label for="published" class="form-label">Published <span class="required">*</span></label>
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
          Schedule
        </button>
      </div>
    </div>
  </form>
</div>
