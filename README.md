https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

{
    "startDate": "2024-11-30T18:30:00.000Z",
    "endDate": "2024-12-10T18:30:00.000Z"
}

download-candidate-list-dialog.ts

import { Component, Inject } from '@angular/core';
import {
  AbstractControl,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';
import { MatDialog,MatDialogRef } from '@angular/material/dialog';
import { AuthService } from '../../services/auth.service';

@Component({
  selector: 'app-download-candidate-list-dialog',
  templateUrl: './download-candidate-list-dialog.component.html',
  styleUrls: ['./download-candidate-list-dialog.component.scss'],
})
export class DownloadCandidateListDialogComponent{
  form: FormGroup;
  get f(): { [key: string]: AbstractControl } {
    return this.form.controls;
  }
  min = new Date();
  timingList: any = [];
  constructor(
    private fb: FormBuilder,
    private dialogRef: MatDialogRef<DownloadCandidateListDialogComponent>,
    public dialog: MatDialog,
    public authService: AuthService
  ) {
    console.log("Inside dialog");
  }
  closeModal() {
    this.dialogRef.close();
  }
  onDateChange(event: any) {
    console.log('on date change', event);
  }

  initForm() {
    this.form = this.fb.group({
      startDate: ['', [Validators.required]],
      endDate: ['', [Validators.required]],
    });
  }

  onSubmit() {
    console.log('THE FORM VALUE', this.form.value);
  }
}

download-candidate-list-dialog.html

<div class="event-edit-modal">
  <div class="event-edit-header">
    <h5 class="heading-text">Edit Event</h5>
    <app-icon icon="close" (click)="closeModal()"></app-icon>
  </div>
  <form class="form-item" [formGroup]="form" (ngSubmit)="onSubmit()">
    <div class="event-edit-body">
      <div class="resume-upload">
        <div class="row">
          <div class="col-lg-12 col-sm-12">
            <div class="form-group form-inner">
              <label class="form-label" for="experiencedPeriod"
                >Start Date <span class="required"></span
              ></label>
              <div class="form-datepicker form-datepicker-custom">
                <input
                  class="form-control"
                  type="text"
                  id="startPicker"
                  formControlName="startDate"
                  placeholder="Choose a date"
                  [matDatepicker]="startPicker"
                  (dateChange)="onDateChange($event.value)"
                />
                <mat-datepicker-toggle
                  matIconSuffix
                  [for]="startPicker"
                ></mat-datepicker-toggle>
              </div>
              <mat-datepicker #startPicker></mat-datepicker>
            </div>
          </div>
          <div class="col-lg-12 col-sm-12">
            <div class="form-group form-inner">
              <label class="form-label" for="experiencedPeriod"
                >End Date <span class="required"></span
              ></label>
              <div class="form-datepicker form-datepicker-custom">
                <input
                  class="form-control"
                  type="text"
                  id="endPicker"
                  formControlName="endDate"
                  placeholder="Choose a date"
                  [matDatepicker]="endPicker"
                  (dateChange)="onDateChange($event.value)"
                />
                <mat-datepicker-toggle
                  matIconSuffix
                  [for]="endPicker"
                ></mat-datepicker-toggle>
              </div>
              <mat-datepicker #endPicker></mat-datepicker>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="event-edit-footer">
      <div class="row justify-content-end">
        <div class="col-lg-3 col-6">
          <button
            type="button"
            (click)="closeModal()"
            class="ags-outline-btn ags-hxl56 ags-padding1624 btn-font16"
          >
            Cancel
          </button>
        </div>
        <div class="col-lg-3 col-6">
          <button
            type="submit"
            (click)="onSubmit"
            class="ags-primary-btn ags-hxl56 ags-padding1624 btn-font16"
          >
            Download Candidate List
          </button>
        </div>
      </div>
    </div>
  </form>
</div>

