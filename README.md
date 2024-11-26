https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-upload.ts

import { Component } from '@angular/core';
import { MatDialog, MatDialogRef } from '@angular/material/dialog';

@Component({
  selector: 'app-req-upload-dialog',
  templateUrl: './req-upload-dialog.component.html',
  styleUrls: ['./req-upload-dialog.component.scss'],
})
export class ReqUploadDialogComponent {
  selectedFile: File | null = null;
  fileError: string | null = null;

  constructor(
    private dialog: MatDialog,
    public dialogRef: MatDialogRef<ReqUploadDialogComponent>
  ) {}

  closeModal() {
    this.dialogRef.close();
  }

  // File selection handler
  onFileSelected(event: Event): void {
    const input = event.target as HTMLInputElement;
    if (input?.files?.length) {
      const file = input.files[0];
      if (file.type !== 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet') {
        this.fileError = 'Please upload a valid .xlsx file.';
        this.selectedFile = null;
      } else {
        this.fileError = null;
        this.selectedFile = file;
      }
    }
  }

  // Form submission handler
  onSubmit(): void {
    if (this.selectedFile) {
      // Handle the file upload logic here
      console.log('File ready for upload:', this.selectedFile);
      // Add your logic for uploading the file to the server here
    }
  }
}


req-upload.html

<div class="req-edit-modal">
  <div class="req-edit-header">
    <h5 class="heading-text">Req Mapping Edit</h5>
    <div>
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>
    </div>
  </div>
  <form (ngSubmit)="onSubmit()">
    <div class="req-edit-body">
      <div class="row">
        <div class="col-lg-6 col-sm-6">
          <div class="form-group ags-form-group">
            <label for="fileUpload" class="form-label">Upload File<span class="required"></span></label>
            <input
              type="file"
              id="fileUpload"
              accept=".xlsx"
              (change)="onFileSelected($event)"
              class="file-upload-input"
            />
            <div *ngIf="fileError" class="error-text">{{ fileError }}</div>
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
          [disabled]="!selectedFile || fileError"
          class="ags-primary-btn ags-hmd44 btn-font16 ags-padding1624"
        >
          Upload
        </button>
      </div>
    </div>
  </form>
</div>
