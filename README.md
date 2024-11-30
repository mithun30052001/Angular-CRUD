https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

remarks-edit-dialog.component.ts

import { Component, OnInit, Inject } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialogRef } from '@angular/material/dialog';
import { MediaMatcher } from '@angular/cdk/layout';
import {
  FormBuilder,
  FormGroup,
  Validators,
  FormControl,
} from '@angular/forms';
import { ReqService } from '@/src/app/shared/services/req.service';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import { AuthService } from '@/src/app/shared/services/auth.service';


const ELEMENT_DATA: any[] = [];

@Component({
  selector: 'app-remarks-edit-dialog',
  templateUrl: './remarks-edit-dialog.component.html',
  styleUrls: ['./remarks-edit-dialog.component.scss'],
})
export class RemarksEditDialogComponent implements OnInit {
  remarkForm: FormGroup;
  data: any;
  form: FormGroup = new FormGroup({
    remarkForm: new FormGroup({
      remarks: new FormControl(''),
    }),
  });
  constructor(
    private dialogRef: MatDialogRef<RemarksEditDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public rowData: any,
    private reqService: ReqService,
    private fb: FormBuilder,
    private applicationService: ApplicationService,
    private authService: AuthService
  ) {
    console.log('MY ROWDATA INSIDE REMARKS', rowData.requesitionId);
    this.remarkForm = this.fb.group({
      remarks: ['', Validators.required],
    });
    this.data = rowData;
  }

  displayedColumns = ['sjoined', 'syto', 'sspoc'];
  dataSource = ELEMENT_DATA;
  queryParams: any;

  closeDialog() {
    this.dialogRef.close();
  }
  ngOnInit(): void {
    this.getRequistionRemarks(this.data?.requisitionId);
    this.onSubmit();
  }

  getRequistionRemarks(reqId: string) {
    this.reqService.getJobRequisitionRemarks(reqId).subscribe(
      (response) => {
        const data = response;
        console.log("RECEIVED RESPONSE", data);
       
        this.dataSource = data.map((item: any) => ({
          name: item.hrName,
          remarks: item.remark,
          createdDate: item.createdDate,
        }));

        console.log('Job requisition updated successfully:', response);
      },
      (error) => {
        console.error('Error updating job requisition:', error);
      }
    );
  }

  onSubmit() {
    if (this.remarkForm.valid) {
      const formData = this.remarkForm.value.remarks;
      console.log('Submitting:', this.remarkForm);
      const requestData = {
        requisitionId: this.data?.requisitionId,
        remarksUpdate: true,
        remarks: formData,
      };
      console.log('My requestData', requestData);
      this.reqService.updateRemarkRequisitions(requestData).subscribe({
        next: (response) => {
          console.log('API Response:', response);
          this.getRequistionRemarks(this.data?.requisitionId);
        },
        error: (error) => {
          console.error('Error:', error);
        },
      });
    } else {
      console.log('Form is invalid');
    }
  }
}

remarks-edit-dialog.component.html

<div class="remarks-modal">
  <div class="remarks-header">
    <h5 class="heading-text">Remarks</h5>
    <div>
      <app-icon icon="close" (click)="closeDialog()"></app-icon>
    </div>
  </div>
  <div class="remarks-body">
    <ng-container>
      <div class="followup-interview">
        <form [formGroup]="remarkForm" (ngSubmit)="onSubmit()">
          <div class="row">
            <div class="col-sm-9">
              <div class="form-group ags-form-group">
                <!-- <input
                  [formControl]="remarks"
                  placeholder="Enter remarks"
                  class="comment form-control"
                /> -->
                <mat-form-field>
                  <input
                    matInput
                    formControlName="remarks"
                    placeholder="Add Remarks"
                    type="text"
                    class="w-100"
                    required
                  />
                </mat-form-field>
              </div>
            </div>
            <div class="col-sm-3">
              <button
                title="Add remarks"
                type="submit"
                class="ags-primary-btn ags-hxl56 ags-padding1624 btn-font16"
              >
                Add
              </button>
            </div>
          </div>
        </form>
        <div class="folowup-table table-responsive mt-3">
          <div class="table-responsive">
            <table mat-table [dataSource]="dataSource">
              <ng-container matColumnDef="sjoined">
                <th mat-header-cell *matHeaderCellDef>Name</th>
                <td mat-cell *matCellDef="let element">
                  {{ element.name }}
                </td>
              </ng-container>
              <ng-container matColumnDef="syto">
                <th mat-header-cell *matHeaderCellDef>Remarks</th>
                <td mat-cell *matCellDef="let element">
                  {{ element.remarks }}
                  <button class="remarks-button">
                    <app-icon icon="small_file"></app-icon>
                  </button>
                </td>
              </ng-container>
              <ng-container matColumnDef="sspoc">
                <th mat-header-cell *matHeaderCellDef>Created date</th>
                <td mat-cell *matCellDef="let element">
                  {{ element.createdDate | date : 'dd MMM YYYY, h:mma' }}
                </td>
              </ng-container>

              <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
              <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
            </table>
          </div>
        </div>
        <!-- <div *ngIf="mobileView" class="folowup-table-mobile mt-3">
          <div>
            <div class="follow-card-mobile" *ngFor="let followup of followups">
              <div class="row">
                <div class="col-6">
                  <h5 class="interview-title">Created date</h5>
                  <p class="follow-content">
                    {{ followup.createdDate | date : 'dd MMM YYYY, h:mma' }}
                  </p>
                </div>
                <div class="col-6">
                  <h5 class="interview-title">Hr name</h5>
                  <p class="follow-content">{{ followup.hrName }}</p>
                </div>
                <div class="col-12">
                  <h5 class="interview-title">Remark</h5>
                  <p class="follow-content" style="word-wrap: break-word">
                    {{ followup.remark }}
                  </p>
                </div>
              </div>
            </div>
          </div>
        </div> -->
      </div>
    </ng-container>
  </div>
  <div class="remarks-footer">
    <div class="row justify-content-end">
      <div class="col-lg-3 col-6">
        <button
          title="Close model"
          (click)="closeDialog()"
          class="ags-primary-btn ags-hxl56 ags-padding1624 btn-font16"
        >
          Close
        </button>
      </div>
    </div>
  </div>
</div>


req.service-method

 getRemarkImage(remarkId: string): Observable<any> {
    const url = `${environment.API_URL}requisition//job/remarks/${remarkId}/image`;
    return this.http.get<any>(url);
  }


