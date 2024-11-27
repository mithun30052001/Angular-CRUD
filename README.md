https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


req.service

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

  updateJobRequisition(data: {
    jobId: string;
    requisitionId: string;
    spoc: string;
    closureDate: string;
    onboardingDate: string;
    manager: string;
    published: boolean;
  }): Observable<any> {
    const url = `${environment.API_URL}requisition/job/create`;
    return this.http.post<any>(url, data);
  }
}


req.mapping.component.ts

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
export class ReqMappingComponent implements OnInit{
  mobileView: any;
  constructor(public dialog: MatDialog, private reqService: ReqService) {}

  displayedColumns = [
    'sjoined',
    'syto',
    'sspoc',
    'opennumber',
    'shift',
    'shift',
    'wfm',
    'openno',
    'action',
  ];
  dataSource = ELEMENT_DATA;
  queryParams: any;

  ngOnInit(): void {
    this.getJobRequisitions();
  }

  getJobRequisitions() {}

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


req.mapping.component.html
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
        <button class="ags-primary-btn ags-hlg48 ags-padding1216 btn-font16"  (click)="reqMappingUpload()">
          Upload Reqs
        </button>
      </div>
    </div>
      <div class="table-responsive">
        <table mat-table [dataSource]="dataSource">
          <ng-container matColumnDef="sjoined">
            <th mat-header-cell *matHeaderCellDef>Rec Id</th>
            <td mat-cell *matCellDef="let element">{{ element.sjoined }}</td>
          </ng-container>
          <ng-container matColumnDef="syto">
            <th mat-header-cell *matHeaderCellDef>Closure Date</th>
            <td mat-cell *matCellDef="let element">{{ element.syto }}</td>
          </ng-container>
          <ng-container matColumnDef="sspoc">
            <th mat-header-cell *matHeaderCellDef>Spoc Name</th>
            <td mat-cell *matCellDef="let element">{{ element.sspoc }}</td>
          </ng-container>
          <ng-container matColumnDef="opennumber">
            <th mat-header-cell *matHeaderCellDef>Manager</th>
            <td mat-cell *matCellDef="let element">
              {{ element.opennumber }}
            </td>
          </ng-container>
          <ng-container matColumnDef="shift">
            <th mat-header-cell *matHeaderCellDef>Onboarding Date</th>
            <td mat-cell *matCellDef="let element">{{ element.shift }}</td>
          </ng-container>

          <ng-container matColumnDef="wfm">
            <th mat-header-cell *matHeaderCellDef>Jobs</th>
            <td mat-cell *matCellDef="let element">{{ element.wfm }}</td>
          </ng-container>
          <ng-container matColumnDef="openno">
            <th mat-header-cell *matHeaderCellDef>Open Numbers</th>
            <td mat-cell *matCellDef="let element">{{ element.openno }}</td>
          </ng-container>
          <ng-container matColumnDef="action">
            <th mat-header-cell *matHeaderCellDef>Action</th>
            <td mat-cell *matCellDef="let element">
              <button class="icon-button" (click)="reqMappingEdit()">
                <app-icon icon="edit_ic" title="Quick edit" />
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

DATA from `${environment.API_URL}requisition/get`
[
    {
        "id": "3ae21603-7afe-40d1-ab93-db8eaedab9ca",
        "createdDate": "2024-11-26T15:17:12.418+00:00",
        "updatedDate": "2024-11-26T15:17:12.418+00:00",
        "active": true,
        "createdBy": null,
        "updatedBy": null,
        "job": null,
        "requestType": "Backfill",
        "requisitionId": "2064",
        "clientName": "BSWH",
        "openNumbers": 15,
        "intentDate": "2024-10-24T00:00:00.000+00:00",
        "closureDate": null,
        "tat": 0,
        "offered": 0,
        "joined": 0,
        "inProgress": 0,
        "yetToOffered": 0,
        "process": "VOICE",
        "location": "Hyderabad",
        "speciality": "PB Collections",
        "jobType": "fresher",
        "shift": "N_5.30 p.m - 2.30 a.m",
        "requisitionStatus": "Open - Not Posted",
        "spoc": null,
        "split": "BSWH",
        "role": "Operation",
        "onboardingDate": null,
        "employeeReferral": 0,
        "direct": 0,
        "vendor": 0,
        "campus": 0,
        "male": 0,
        "female": 0,
        "manager": null,
        "published": false,
        "remarks": null,
        "monthToBeOnboarded": null
    },
    {
        "id": "b13250f8-cfbe-44df-822b-44347e0cb673",
        "createdDate": "2024-11-26T15:17:12.416+00:00",
        "updatedDate": "2024-11-26T15:17:12.416+00:00",
        "active": true,
        "createdBy": null,
        "updatedBy": null,
        "job": null,
        "requestType": "New Position/Additional Job Role",
        "requisitionId": "2065",
        "clientName": "AGS",
        "openNumbers": 1,
        "intentDate": "2024-10-25T00:00:00.000+00:00",
        "closureDate": null,
        "tat": 0,
        "offered": 0,
        "joined": 0,
        "inProgress": 0,
        "yetToOffered": 0,
        "process": "VOICE",
        "location": "Bangalore",
        "speciality": "Support",
        "jobType": "experienced",
        "shift": "D_8.00 a.m - 5.00 p.m",
        "requisitionStatus": "Open - Not Posted",
        "spoc": null,
        "split": "NA",
        "role": "Support",
        "onboardingDate": null,
        "employeeReferral": 0,
        "direct": 0,
        "vendor": 0,
        "campus": 0,
        "male": 0,
        "female": 0,
        "manager": null,
        "published": false,
        "remarks": null,
        "monthToBeOnboarded": null
    },
    {
        "id": "de6203ee-482a-4640-95b5-25c3d09edb2b",
        "createdDate": "2024-11-26T15:17:12.402+00:00",
        "updatedDate": "2024-11-26T15:17:12.402+00:00",
        "active": true,
        "createdBy": null,
        "updatedBy": null,
        "job": null,
        "requestType": "New Position/Additional Job Role",
        "requisitionId": "2066",
        "clientName": "AGS",
        "openNumbers": 1,
        "intentDate": "2024-10-25T00:00:00.000+00:00",
        "closureDate": null,
        "tat": 0,
        "offered": 0,
        "joined": 0,
        "inProgress": 0,
        "yetToOffered": 0,
        "process": "VOICE",
        "location": "Bangalore",
        "speciality": "Support",
        "jobType": "experienced",
        "shift": "D_8.00 a.m - 5.00 p.m",
        "requisitionStatus": "Open - Not Posted",
        "spoc": null,
        "split": "NA",
        "role": "Support",
        "onboardingDate": null,
        "employeeReferral": 0,
        "direct": 0,
        "vendor": 0,
        "campus": 0,
        "male": 0,
        "female": 0,
        "manager": null,
        "published": false,
        "remarks": null,
        "monthToBeOnboarded": null
    }
]
