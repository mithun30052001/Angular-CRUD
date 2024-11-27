https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

interface

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


req-mapping.ts

import { Component, OnInit } from '@angular/core';
import { MatIconModule } from '@angular/material/icon';
import { MatTableModule } from '@angular/material/table';
import { SharedModule } from '../../../shared/shared.module';
import { MatDialog } from '@angular/material/dialog';
import { ReqMappingEditDialogComponent } from '@/src//app/hr/components/req-mapping-edit-dialog/req-mapping-edit-dialog.component';
import { ReqUploadDialogComponent } from '../req-upload-dialog/req-upload-dialog.component';
import { ReqService } from '@/src/app/shared/services/req.service';
import { Observable } from 'rxjs';
import { Requisition } from './req-mapping.component'; // import the interface

@Component({
  selector: 'app-req-mapping',
  templateUrl: './req-mapping.component.html',
  styleUrls: ['./req-mapping.component.scss'],
  standalone: true,
  imports: [MatTableModule, MatIconModule, SharedModule],
})
export class ReqMappingComponent implements OnInit {
  mobileView: any;
  requisitions: Requisition[] = []; // Store fetched requisition data

  // Define the columns to display in the table
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

  constructor(
    public dialog: MatDialog,
    private reqService: ReqService
  ) {}

  ngOnInit(): void {
    this.getJobRequisitions();
  }

  // Method to fetch requisition data from the service
  getJobRequisitions() {
    this.reqService.getRequisitions().subscribe({
      next: (data: Requisition[]) => {
        this.requisitions = data; // Assign data to requisitions variable
      },
      error: (err) => {
        console.error('Error fetching requisitions', err);
      }
    });
  }

  reqMappingEdit() {
    console.log('Opening Edit Dialog');
    this.dialog.open(ReqMappingEditDialogComponent, {
      maxWidth: this.mobileView ? '500px' : '100%',
      width: this.mobileView ? '100%' : '50%',
      height: 'auto',
      ...(this.mobileView && { position: { bottom: '5px' } }),
      closeOnNavigation: true,
      disableClose: false,
    });
  }

  reqMappingUpload() {
    console.log('Opening Upload Dialog');
    this.dialog.open(ReqUploadDialogComponent, {
      maxWidth: this.mobileView ? '700px' : '100%',
      width: this.mobileView ? '100%' : '50%',
      height: 'auto',
      ...(this.mobileView && { position: { bottom: '5px' } }),
      closeOnNavigation: true,
      disableClose: false,
    });
  }
}


req mapping.html

<div class="hr-vendor-header">
  <!-- Optional Header (breadcrumb) -->
</div>

<div class="hr-vendor-list">
  <div class="hr-vendor-list-body">
    <div class="row">
      <!-- Vendor count blocks (keep as it is for now) -->
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
          <!-- Request Type Column -->
          <ng-container matColumnDef="requestType">
            <th mat-header-cell *matHeaderCellDef>Request Type</th>
            <td mat-cell *matCellDef="let element">{{ element.requestType }}</td>
          </ng-container>

          <!-- Requisition ID Column -->
          <ng-container matColumnDef="requisitionId">
            <th mat-header-cell *matHeaderCellDef>Requisition ID</th>
            <td mat-cell *matCellDef="let element">{{ element.requisitionId }}</td>
          </ng-container>

          <!-- Process Column -->
          <ng-container matColumnDef="process">
            <th mat-header-cell *matHeaderCellDef>Process</th>
            <td mat-cell *matCellDef="let element">{{ element.process }}</td>
          </ng-container>

          <!-- Client Name Column -->
          <ng-container matColumnDef="clientName">
            <th mat-header-cell *matHeaderCellDef>Client</th>
            <td mat-cell *matCellDef="let element">{{ element.clientName }}</td>
          </ng-container>

          <!-- Speciality Column -->
          <ng-container matColumnDef="speciality">
            <th mat-header-cell *matHeaderCellDef>Speciality</th>
            <td mat-cell *matCellDef="let element">{{ element.speciality }}</td>
          </ng-container>

          <!-- Job Type Column -->
          <ng-container matColumnDef="jobType">
            <th mat-header-cell *matHeaderCellDef>Job Type</th>
            <td mat-cell *matCellDef="let element">{{ element.jobType }}</td>
          </ng-container>

          <!-- Location Column -->
          <ng-container matColumnDef="location">
            <th mat-header-cell *matHeaderCellDef>Location</th>
            <td mat-cell *matCellDef="let element">{{ element.location }}</td>
          </ng-container>

          <!-- Shift Column -->
          <ng-container matColumnDef="shift">
            <th mat-header-cell *matHeaderCellDef>Shift</th>
            <td mat-cell *matCellDef="let element">{{ element.shift }}</td>
          </ng-container>

          <!-- Open Numbers Column -->
          <ng-container matColumnDef="openNumbers">
            <th mat-header-cell *matHeaderCellDef>Open Numbers</th>
            <td mat-cell *matCellDef="let element">{{ element.openNumbers }}</td>
          </ng-container>

          <!-- Action Column -->
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



req-service.ts

import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
import { environment } from '@/environments/environment';
import { Requisition } from '@/app/hr/components/req-mapping/req-mapping.component'; // Import the Requisition interface

@Injectable({
  providedIn: 'root',
})
export class ReqService {
  constructor(private http: HttpClient) {}

  // Fetch the requisitions data
  getRequisitions(): Observable<Requisition[]> {
    const url = `${environment.API_URL}requisition/get`;
    return this.http.get<Requisition[]>(url);
  }
}
