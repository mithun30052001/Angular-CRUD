https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-mapping-edit-dialog.ts

import { Component, Inject } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from 'path-to-req-service';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;

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

req-mapping-edit.html

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


req-service.ts

import { environment } from '@/environments/environment';
import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ReqService {
  constructor(private http: HttpClient) {}

  // Existing uploadReqdetails method
  uploadReqdetails(file: File): Observable<any> {
    const formData = new FormData();
    formData.append('file', file, file.name);

    const url = `${environment.API_URL}/job-services/requisition/upload`;

    return this.http.post<any>(url, formData);
  }

  // New method to update job requisition
  updateJobRequisition(data: {
    jobId: string;
    requisitionId: string;
    spoc: string;
    closureDate: string;
    onboardingDate: string;
    openNumbers: number;
  }): Observable<any> {
    const url = `${environment.API_URL}/requisition/job/update`;
    return this.http.put<any>(url, data);
  }
}


form-config

export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;

  formFields = [
    { name: 'jobId', label: 'Job ID', placeholder: 'Enter job ID', type: 'text' },
    { name: 'reqId', label: 'Requisition ID', placeholder: 'Enter requisition ID', type: 'text' },
    { name: 'spoc', label: 'SPOC', placeholder: 'Enter SPOC name', type: 'text' },
    { name: 'closureDate', label: 'Closure Date', placeholder: 'Enter closure date', type: 'date' },
    { name: 'onboardingDate', label: 'Onboarding Date', placeholder: 'Enter onboarding date', type: 'date' },
    { name: 'openNumbers', label: 'Open Numbers', placeholder: 'Enter open numbers', type: 'number' }
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
