https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


join date is now coming as joinedDate
: "2025-01-16T18:30:00.000Z" but i want the joindate as "joinDate":"2025-01-17",

shortlist-candidate-dialog.ts

import { Component, Inject, OnInit } from '@angular/core';
import { AbstractControl, FormBuilder, FormGroup, Validators } from '@angular/forms';
import { MAT_DIALOG_DATA, MatDialogRef } from '@angular/material/dialog';

@Component({
  selector: 'app-shortlist-candidate-dialog',
  templateUrl: './shortlist-candidate-dialog.component.html',
  styleUrls: ['./shortlist-candidate-dialog.component.scss'],
})
export class ShortlistCandidateDialogComponent implements OnInit {
  form: FormGroup;
  minDate = new Date();

  constructor(
    private fb: FormBuilder,
    public dialogRef: MatDialogRef<ShortlistCandidateDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public data: any
  ) {}

  ngOnInit(): void {
    this.initForm();
  }

  initForm() {
    // Initialize form with requisitionId prefilled if available, and joinedDate empty
    this.form = this.fb.group({
      requisitionId: [{ value: this.data.requisitionId || '', disabled: !!this.data.requisitionId }, [Validators.required]],
      joinedDate: ['', Validators.required]
    });
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.form.valid) {
      const formValues = this.form.value;
      this.dialogRef.close(formValues); // Pass the form values back to the parent component
    }
  }

  get f() {
    return this.form.controls;
  }
}



shortlist-candidate-dialog.component.html

<div class="shortlist-dialog">
  <div class="header">
    <h5 class="heading-text">Shortlist Candidate</h5>
    <app-icon icon="close" (click)="closeModal()"></app-icon>
  </div>
  <form [formGroup]="form" (ngSubmit)="onSubmit()">
    <div class="body">
      <div class="form-group">
        <label for="requisitionId">Requisition ID</label>
        <input
          formControlName="requisitionId"
          id="requisitionId"
          type="text"
          class="form-control"
          [readonly]="true"
        />
      </div>
      <div class="form-group">
        <label for="joinedDate">Joined Date</label>
        <input
          formControlName="joinedDate"
          matDatepicker
          [min]="minDate"
          placeholder="Choose a date"
          class="form-control"
          id="joinedDate"
        />
        <mat-datepicker-toggle matIconSuffix [for]="joinedDatePicker"></mat-datepicker-toggle>
        <mat-datepicker #joinedDatePicker></mat-datepicker>
      </div>
    </div>
    <div class="footer">
      <button type="button" class="cancel-btn" (click)="closeModal()">Cancel</button>
      <button type="submit" class="shortlist-btn">Shortlist Candidate</button>
    </div>
  </form>
</div>



case in candidate-view

const dialogRef = this.dialog.open(ShortlistCandidateDialogComponent, {
      width: this.mobileView ? '100%' : '30%',
      maxWidth: this.mobileView ? '500px' : '100%',
      data: {
        requisitionId: this.profile?.requisitionId || '' // Pass the requisitionId if available
      }
    });

    dialogRef.afterClosed().subscribe(result => {
      if (result) {
        // API call to update application status
        const { requisitionId, joinedDate } = result;
        const payload = {
          status: 'SHORTLISTED',
          requisitionId: requisitionId,
          joinedDate: joinedDate
        };

        this.applicationService.updateApplicationStatus(this.applicationId, payload).then(() => {
          // Do any follow-up logic (e.g., refresh data)
          this.getApplication();
        }).catch(err => {
          console.error('Error updating application status', err);
        });
      }
    });
