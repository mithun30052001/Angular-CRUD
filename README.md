https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

Add columns :  RecId closuredate spocname manager onboardingdate jobid opennumbers

req-mapping-dialog.ts

import { Component } from '@angular/core';
import { MatDialog, MatDialogRef } from '@angular/material/dialog';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent {
  constructor(
    private dialog: MatDialog,
    public dialogRef: MatDialogRef<ReqMappingEditDialogComponent>
  ) {}
  closeModal() {
    this.dialogRef.close();
  }
}

req-mapping-dialog.html

<div class="req-edit-modal">
  <div class="req-edit-header">
    <h5 class="heading-text">Req Mapping Edit</h5>
    <div>
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>
    </div>
  </div>
  <form>
    <div class="req-edit-body">
      <div class="row">
        <div class="col-lg-6 col-sm-6">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Job<span class="required"></span
            ></label>
            <mat-form-field>
              <input
                type="text"
                placeholder="search jobs"
                aria-label="string"
                matInput
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
        >
          Schedule
        </button>
      </div>
    </div>
  </form>
</div>

req-service.ts

import { environment } from '@/environments/environment';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class ReqService {
  constructor(private http: HttpClient) {}

  uploadReqdetails(file: File): Observable<any> {
    const formData = new FormData();
    formData.append('file', file, file.name);

    const url = `${environment.API_URL}/job-services/requisition/upload`;

    return this.http.post<any>(url, formData);
  }
}


put call to  '`${environment.API_URL}requisition/job/update' with

{
"jobId"
"requisitionId"
"spoc"
"closureDate"
"onBoardingDate"
"openNumbers"
}

