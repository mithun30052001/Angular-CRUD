https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


req-mapping.component.ts

import { Component, OnInit } from '@angular/core';
import { MatIconModule } from '@angular/material/icon';
import { MatTableModule } from '@angular/material/table';
import { SharedModule } from '../../../shared/shared.module';
import { MatDialog } from '@angular/material/dialog';
import { ReqMappingEditDialogComponent } from '@/src//app/hr/components/req-mapping-edit-dialog/req-mapping-edit-dialog.component';
import { ReqUploadDialogComponent } from '../req-upload-dialog/req-upload-dialog.component';
import { ReqService } from '@/src/app/shared/services/req.service';
export interface PeriodicElement {
  sjoined: string;
  syto: string;
  sspoc: string;
  opennumber: string;
  shift: string;
  wfm: string;
  openno: string;
  action: string;
}

export interface Requisition {
  id: string;
  createdDate: string;
  updatedDate: string;
  active: boolean;
  requestType: string;
  requisitionId: string;
  clientName: string;
  openNumbers: number;
  intentDate: string;
  closureDate: string | null;
  process: string;
  location: string;
  speciality: string;
  jobType: string;
  shift: string;
  requisitionStatus: string;
  spoc: string | null;
  split: string;
  role: string;
  onboardingDate: string | null;
  published: boolean;
}


const ELEMENT_DATA: PeriodicElement[] = [
  {
    sjoined: '20',
    syto: '20',
    sspoc: '20',
    opennumber: '20',
    shift: '20',
    wfm: 'eee',
    openno: 'eeee',
    action: '',
  },
];
@Component({
  selector: 'app-req-mapping',
  templateUrl: './req-mapping.component.html',
  styleUrls: ['./req-mapping.component.scss'],
  standalone: true,
  imports: [MatTableModule, MatIconModule, SharedModule],
})
export class ReqMappingComponent implements OnInit {
  mobileView: any;
  requisitions: Requisition[] = [];

  constructor(public dialog: MatDialog, private reqService: ReqService) {}

  displayedColumns = [
    'requestType',
    'requisitionId',
    'process',
    'clientName',
    'speciality',
    'jobType',
    'location',
    'shift',
    'openNumbers',
    'action',
  ];

  dataSource = ELEMENT_DATA;
  queryParams: any;

  ngOnInit(): void {
    this.getJobRequisitions();
  }

  getJobRequisitions() {
    this.reqService.getRequisitions().subscribe({
      next: (data: Requisition[]) => {
        this.requisitions = data;
      },
      error: (err) => {
        console.error('Error fetching requisitions', err);
      },
    });
  }

  reqMappingEdit() {
    console.log('amhsdbhasbdhasbdfb');
    this.dialog.open(ReqMappingEditDialogComponent, {
      maxWidth: this.mobileView ? '500px' : '100%',
      width: this.mobileView ? '100%' : '50%',
      height: 'auto',
      ...(this.mobileView && {
        position: { bottom: '5px' },
      }),
      closeOnNavigation: true,
      disableClose: false,
    });
  }

  reqMappingUpload() {
    console.log('amhsdbhasbdhasbdfb');
    this.dialog.open(ReqUploadDialogComponent, {
      maxWidth: this.mobileView ? '700px' : '100%',
      width: this.mobileView ? '100%' : '50%',
      height: 'auto',
      ...(this.mobileView && {
        position: { bottom: '5px' },
      }),
      closeOnNavigation: true,
      disableClose: false,
    });
  }
}


req-mapping.component.html

<div class="hr-vendor-header">
  <!-- <app-breadcrumb
    [pageName]="queryParams.name"
    header="Hiring Update"
    backNavigation="h/hiring-update/"
  ></app-breadcrumb> -->
</div>
<div class="hr-vendor-list">
  <div class="hr-vendor-list-body">
    <div class="row">
      <div class="col-md-12 col-lg-3 col-xl-4 col-xxl-4 col-xs-12">
        <div class="ags-count-vendor">
          <h4>Vendor count</h4>
          <p class="pt-4">30</p>
        </div>
      </div>
      <div class="col-md-12 col-lg-5 col-xl-4 col-xxl-4 col-xs-12">
        <div class="ags-count-vendor">
          <h4 class="me-auto">Offered</h4>
          <p class="pt-4">30</p>
        </div>
      </div>
      <div class="col-md-12 col-lg-4 col-xl-4 col-xxl-4 col-xs-12">
        <div class="ags-count-vendor">
          <h4 class="me-auto">Joined</h4>
          <p class="pt-4">30</p>
        </div>
      </div>
    </div>

    <div class="vendor-task-table">
      <div class="vendor-task-table-header">
        <div class="row d-flex align-items-center">
          <div class="col-sm-4 col-lg-7 col-xl-7">
            <h3>Req Mapping</h3>
          </div>
        </div>
      </div>

      <div class="row justify-content-end mb-2">
        <div class="col-sm-2">
          <button class="ags-primary-btn ags-hlg48 ags-padding1216 btn-font16" (click)="reqMappingUpload()">
            Upload Reqs
          </button>
        </div>
      </div>

      <div class="table-responsive">
        <table mat-table [dataSource]="requisitions">
          <ng-container matColumnDef="requestType">
            <th mat-header-cell *matHeaderCellDef>Request Type</th>
            <td mat-cell *matCellDef="let element">{{ element.requestType }}</td>
          </ng-container>

          <ng-container matColumnDef="requisitionId">
            <th mat-header-cell *matHeaderCellDef>Requisition ID</th>
            <td mat-cell *matCellDef="let element">{{ element.requisitionId }}</td>
          </ng-container>

          <ng-container matColumnDef="process">
            <th mat-header-cell *matHeaderCellDef>Process</th>
            <td mat-cell *matCellDef="let element">{{ element.process }}</td>
          </ng-container>

          <ng-container matColumnDef="clientName">
            <th mat-header-cell *matHeaderCellDef>Client</th>
            <td mat-cell *matCellDef="let element">{{ element.clientName }}</td>
          </ng-container>

          <ng-container matColumnDef="speciality">
            <th mat-header-cell *matHeaderCellDef>Speciality</th>
            <td mat-cell *matCellDef="let element">{{ element.speciality }}</td>
          </ng-container>

          <ng-container matColumnDef="jobType">
            <th mat-header-cell *matHeaderCellDef>Job Type</th>
            <td mat-cell *matCellDef="let element">{{ element.jobType }}</td>
          </ng-container>

          <ng-container matColumnDef="location">
            <th mat-header-cell *matHeaderCellDef>Location</th>
            <td mat-cell *matCellDef="let element">{{ element.location }}</td>
          </ng-container>

          <ng-container matColumnDef="shift">
            <th mat-header-cell *matHeaderCellDef>Shift</th>
            <td mat-cell *matCellDef="let element">{{ element.shift }}</td>
          </ng-container>

          <ng-container matColumnDef="openNumbers">
            <th mat-header-cell *matHeaderCellDef>Open Numbers</th>
            <td mat-cell *matCellDef="let element">{{ element.openNumbers }}</td>
          </ng-container>

          <ng-container matColumnDef="action">
            <th mat-header-cell *matHeaderCellDef>Actions</th>
            <td mat-cell *matCellDef="let element">
              <button mat-icon-button (click)="reqMappingEdit()">
                <mat-icon>edit</mat-icon>
              </button>
            </td>
          </ng-container>

          <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
          <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
        </table>
      </div>
    </div>
  </div>
</div>

req-mapping-edit-dialog.component.ts

import { Component } from '@angular/core';
import { MatDialogRef } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent {
  reqForm: FormGroup;

  formFields = [
    {
      name: 'jobId',
      label: 'Job ID',
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
    private reqService: ReqService,
    private fb: FormBuilder
  ) {
    this.reqForm = this.fb.group({
      jobId: ['', Validators.required],
      requisitionId: ['', Validators.required],
      spoc: ['', Validators.required],
      closureDate: ['', Validators.required],
      onboardingDate: ['', Validators.required],
      manager: ['', Validators.required], // New manager field with validation
      published: [true, Validators.required], // New published radio button field with default value true
    });
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
          this.dialogRef.close(true); // Close dialog on success
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
        >
          Publish
        </button>
      </div>
    </div>
  </form>
</div>


