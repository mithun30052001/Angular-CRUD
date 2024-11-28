https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


hiring.ts

import { MatIconModule } from '@angular/material/icon';
import { MatTableDataSource, MatTableModule } from '@angular/material/table';
import { ApplicationService } from '@/src/app/shared/services/job-application.service';
import { SharedModule } from '../../../shared/shared.module';
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';
import { ReqService } from '@/src/app/shared/services/req.service';
import { RemarksEditDialogComponent } from '@/src/app/hr/components/remarks-edit-dialog/remarks-edit-dialog.component';
import { MatDialog } from '@angular/material/dialog';

export interface PeriodicElement {
  name: string;
  position: string;
  weight: string;
  symbol: string;
  smale: string;
  sfemal: string;
  closure: string;
  closuretat: string;
  remark: string;
  svendor: string;
  sjoined: string;
  syto: string;
  sspoc: string;
  opennumber: string;
  shift: string;
  wfm: string;
  openno: string;
  spoc: string;
  onboarding: string;
  joined: string;
  offered: string;
  yto: string;
  wip: string;
  pipeline: string;
  ers: string;
  direct: string;
  vendor: string;
  male: string;
  female: string;
  remarks: string;
  actions: string;
}

const ELEMENT_DATA: PeriodicElement[] = [
  {
    name: '1907',
    position: 'Growth Existing',
    weight: '2-Sep-24',
    symbol: '25-Nov-24',
    smale: '84',
    sfemal: 'Insurance Eligibility & Benefits Verification',
    closure: 'NA',
    closuretat: 'NA',
    remark: 'Barnabas',
    svendor: 'Barnabas',
    sjoined: 'Production',
    syto: 'Voice',
    sspoc: 'Lateral',
    opennumber: 'Chennai',
    shift: 'Night',
    wfm: 'Oct 24',
    openno: '25',
    spoc: 'Arun',
    onboarding: '4-Nov-24',
    joined: '26',
    offered: '234',
    yto: '456',
    wip: '-1',
    pipeline: '234',
    ers: '987',
    direct: '345',
    vendor: '345',
    male: '676',
    female: '223',
    remarks:
      'Your attention to detail and commitment to excellence are clearly',
    actions: '',
  },
  {
    name: '1907',
    position: 'Growth Existing',
    weight: '2-Sep-24',
    symbol: '25-Nov-24',
    smale: '84',
    sfemal: 'Insurance Eligibility & Benefits Verification',
    closure: 'NA',
    closuretat: 'NA',
    remark: 'Barnabas',
    svendor: 'Barnabas',
    sjoined: 'Production',
    syto: 'Voice',
    sspoc: 'Lateral',
    opennumber: 'Chennai',
    shift: 'Night',
    wfm: 'Oct 24',
    openno: '25',
    spoc: 'Arun',
    onboarding: '4-Nov-24',
    joined: '26',
    offered: '234',
    yto: '456',
    wip: '-1',
    pipeline: '234',
    ers: '987',
    direct: '345',
    vendor: '345',
    male: '676',
    female: '223',
    remarks:
      'Your attention to detail and commitment to excellence are clearly',
    actions: '',
  },
  {
    name: '1907',
    position: 'Growth Existing',
    weight: '2-Sep-24',
    symbol: '25-Nov-24',
    smale: '84',
    sfemal: 'Insurance Eligibility & Benefits Verification',
    closure: 'NA',
    closuretat: 'NA',
    remark: 'Barnabas',
    svendor: 'Barnabas',
    sjoined: 'Production',
    syto: 'Voice',
    sspoc: 'Lateral',
    opennumber: 'Chennai',
    shift: 'Night',
    wfm: 'Oct 24',
    openno: '25',
    spoc: 'Arun',
    onboarding: '4-Nov-24',
    joined: '26',
    offered: '234',
    yto: '456',
    wip: '-1',
    pipeline: '234',
    ers: '987',
    direct: '345',
    vendor: '345',
    male: '676',
    female: '223',
    remarks:
      'Your attention to detail and commitment to excellence are clearly',
    actions: '',
  },
  {
    name: '1907',
    position: 'Growth Existing',
    weight: '2-Sep-24',
    symbol: '25-Nov-24',
    smale: '84',
    sfemal: 'Insurance Eligibility & Benefits Verification',
    closure: 'NA',
    closuretat: 'NA',
    remark: 'Barnabas',
    svendor: 'Barnabas',
    sjoined: 'Production',
    syto: 'Voice',
    sspoc: 'Lateral',
    opennumber: 'Chennai',
    shift: 'Night',
    wfm: 'Oct 24',
    openno: '25',
    spoc: 'Arun',
    onboarding: '4-Nov-24',
    joined: '26',
    offered: '234',
    yto: '456',
    wip: '-1',
    pipeline: '234',
    ers: '987',
    direct: '345',
    vendor: '345',
    male: '676',
    female: '223',
    remarks:
      'Your attention to detail and commitment to excellence are clearly',
    actions: '',
  },
  {
    name: '1907',
    position: 'Growth Existing',
    weight: '2-Sep-24',
    symbol: '25-Nov-24',
    smale: '84',
    sfemal: 'Insurance Eligibility & Benefits Verification',
    closure: 'NA',
    closuretat: 'NA',
    remark: 'Barnabas',
    svendor: 'Barnabas',
    sjoined: 'Production',
    syto: 'Voice',
    sspoc: 'Lateral',
    opennumber: 'Chennai',
    shift: 'Night',
    wfm: 'Oct 24',
    openno: '25',
    spoc: 'Arun',
    onboarding: '4-Nov-24',
    joined: '26',
    offered: '234',
    yto: '456',
    wip: '-1',
    pipeline: '234',
    ers: '987',
    direct: '345',
    vendor: '345',
    male: '676',
    female: '223',
    remarks:
      'Your attention to detail and commitment to excellence are clearly',
    actions: '',
  },
  {
    name: '1907',
    position: 'Growth Existing',
    weight: '2-Sep-24',
    symbol: '25-Nov-24',
    smale: '84',
    sfemal: 'Insurance Eligibility & Benefits Verification',
    closure: 'NA',
    closuretat: 'NA',
    remark: 'Barnabas',
    svendor: 'Barnabas',
    sjoined: 'Production',
    syto: 'Voice',
    sspoc: 'Lateral',
    opennumber: 'Chennai',
    shift: 'Night',
    wfm: 'Oct 24',
    openno: '25',
    spoc: 'Arun',
    onboarding: '4-Nov-24',
    joined: '26',
    offered: '234',
    yto: '456',
    wip: '-1',
    pipeline: '234',
    ers: '987',
    direct: '345',
    vendor: '345',
    male: '676',
    female: '223',
    remarks:
      'Your attention to detail and commitment to excellence are clearly',
    actions: '',
  },
];

@Component({
  selector: 'app-hiring-update',
  templateUrl: './hiring-update.component.html',
  styleUrls: ['./hiring-update.component.scss'],
  standalone: true,
  imports: [MatTableModule, MatIconModule, SharedModule],
})
export class HiringUpdateComponent {
  mobileView: any;
  // hiringList: any[] = [];

  constructor(
    public dialog: MatDialog,
    private router: Router,
    private reqService: ReqService
  ) {}

  navigateReqMapping() {
    this.router.navigate([`h/req-mapping`], {});
  }

  displayedColumns = [
    'name',
    'position',
    'weight',
    'symbol',
    'star',
    'smale',
    'closure',
    'closuretat',
    'sfemal',
    'remark',
    'svendor',
    'sjoined',
    'syto',
    'sspoc',
    'opennumber',
    'shift',
    'shift',
    'wfm',
    'openno',
    'spoc',
    'onboarding',
    'joined',
    'offered',
    'yto',
    'wip',
    'pipeline',
    'ers',
    'direct',
    'vendor',
    'male',
    'female',
    'remarks',
    'actions',
  ];
  dataSource = ELEMENT_DATA;
  requisitionParams = {
    startDate: '2024-11-18',
    endDate: '2024-11-27',
    page: 0,
    size: 10,
    pageable: true,
  };

  ngOnInit() {
    this.getHiring();
  }

  getHiring() {
    this.reqService.getPublishedRequisitions(this.requisitionParams).subscribe(
      (response) => {
        console.log('Job requisition updated successfully:', response.response);
      },
      (error) => {
        console.error('Error updating job requisition:', error);
      }
    );
  }

  remarksEditDialog() {
    this.dialog.open(RemarksEditDialogComponent, {
      maxWidth: this.mobileView ? '500px' : '100%',
      height: 'auto',
      width: this.mobileView ? '100%' : '60%',
      ...(this.mobileView && {
        height: 'auto',
        position: { bottom: '0' },
      }),
      closeOnNavigation: true,
      disableClose: true,
    });
  }
}


hiring.html

<div class="hiring-update-full">
  <div class="listing-content">
    <div class="row justify-content-end mb-2">
      <div class="col-sm-2">
        <button
          class="ags-primary-btn ags-hlg48 ags-padding1216 btn-font16"
          (click)="navigateReqMapping()"
        >
          Req mapping
        </button>
      </div>
    </div>
    <div class="candidate-task-table table-responsive">
      <section class="example-container mat-elevation-z8" tabindex="0">
        <table mat-table [dataSource]="dataSource">
          <!-- Name Column -->
          <!-- <ng-container
            matColumnDef="column"
            *ngFor="let column of displayedColumns"
          > -->
          <ng-container matColumnDef="name" sticky>
            <th mat-header-cell *matHeaderCellDef>Request Type</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.name }}</p>
            </td>
          </ng-container>

          <!-- Position Column -->
          <ng-container matColumnDef="position" sticky>
            <th mat-header-cell *matHeaderCellDef>Oracle-req ID</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.position }}</p>
            </td>
          </ng-container>

          <!-- Weight Column -->
          <ng-container matColumnDef="weight" sticky>
            <th mat-header-cell *matHeaderCellDef>Indent Date</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.weight }}</p>
            </td>
          </ng-container>

          <!-- Symbol Column -->
          <ng-container matColumnDef="symbol" sticky>
            <th mat-header-cell *matHeaderCellDef>Closure Date</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.symbol }}</p>
            </td>
          </ng-container>

          <ng-container matColumnDef="smale" sticky>
            <th mat-header-cell *matHeaderCellDef>TAT</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.smale }}</p>
            </td>
          </ng-container>
          <!-- <ng-container matColumnDef="star" stickyEnd>
            <th mat-header-cell *matHeaderCellDef aria-label="row actions">
              &nbsp;
            </th>
            <td mat-cell *matCellDef="let element">
              <mat-icon>more_vert</mat-icon>asdsdsad
            </td>
          </ng-container> -->
          <ng-container matColumnDef="closure">
            <th mat-header-cell *matHeaderCellDef>Revised Closure Date</th>
            <td mat-cell *matCellDef="let element">
              <p title="{{ element.closure }}">{{ element.closure }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="closuretat">
            <th mat-header-cell *matHeaderCellDef>Revised Closure Date- TAT</th>
            <td mat-cell *matCellDef="let element">
              <p title="{{ element.closuretat }}">{{ element.closuretat }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="sfemal">
            <th mat-header-cell *matHeaderCellDef>Process</th>
            <td mat-cell *matCellDef="let element">
              <p title="{{ element.sfemal }}">{{ element.sfemal }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="remark">
            <th mat-header-cell *matHeaderCellDef>Client</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.remark }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="svendor">
            <th mat-header-cell *matHeaderCellDef>Split</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.svendor }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="sjoined">
            <th mat-header-cell *matHeaderCellDef>Role</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.sjoined }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="syto">
            <th mat-header-cell *matHeaderCellDef>Speciality</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.syto }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="sspoc">
            <th mat-header-cell *matHeaderCellDef>Job Type</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.sspoc }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="opennumber">
            <th mat-header-cell *matHeaderCellDef>Location</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.opennumber }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="shift">
            <th mat-header-cell *matHeaderCellDef>Shift</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.shift }}</p>
            </td>
          </ng-container>

          <ng-container matColumnDef="wfm">
            <th mat-header-cell *matHeaderCellDef>WFM</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.wfm }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="openno">
            <th mat-header-cell *matHeaderCellDef>Pending - TA</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.openno }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="spoc">
            <th mat-header-cell *matHeaderCellDef>Spoc</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.spoc }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="onboarding">
            <th mat-header-cell *matHeaderCellDef>Onboarded</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.onboarding }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="joined">
            <th mat-header-cell *matHeaderCellDef>Joined</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.joined }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="offered">
            <th mat-header-cell *matHeaderCellDef>Offered</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.offered }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="yto">
            <th mat-header-cell *matHeaderCellDef>YTO</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.yto }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="wip">
            <th mat-header-cell *matHeaderCellDef>WIP</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.wip }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="pipeline">
            <th mat-header-cell *matHeaderCellDef>Pipeline</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.pipeline }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="ers">
            <th mat-header-cell *matHeaderCellDef>ERS</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.ers }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="direct">
            <th mat-header-cell *matHeaderCellDef>Direct</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.direct }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="vendor">
            <th mat-header-cell *matHeaderCellDef>Vendor</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.vendor }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="campus">
            <th mat-header-cell *matHeaderCellDef>Campus</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.campus }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="male">
            <th mat-header-cell *matHeaderCellDef>Male</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.male }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="female">
            <th mat-header-cell *matHeaderCellDef>Female</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.female }}</p>
            </td>
          </ng-container>
          <!-- <ng-container matColumnDef="referrals">
            <th mat-header-cell *matHeaderCellDef>Remarks</th>
            <td mat-cell *matCellDef="let element">
              <p>{{ element.referrals }}</p>
            </td>
          </ng-container> -->
          <ng-container matColumnDef="manager">
            <th mat-header-cell *matHeaderCellDef>Manager</th>
            <td mat-cell *matCellDef="let element">
              <p title="{{ element.manager }}">{{ element.manager }}</p>
            </td>
          </ng-container>
          <ng-container matColumnDef="remarks">
            <th mat-header-cell *matHeaderCellDef>Remarks</th>
            <td mat-cell *matCellDef="let element" class="d-flex">
              <p title="{{ element.remarks }}">{{ element.remarks }}</p>
              <button
                class="icon-button"
                title="Edit remarks"
                (click)="remarksEditDialog()"
              >
                <app-icon icon="info"></app-icon>
              </button>
            </td>
          </ng-container>
          <ng-container matColumnDef="actions">
            <th mat-header-cell *matHeaderCellDef>Actions</th>
            <td mat-cell *matCellDef="let element">
              <button class="icon-button">
                <app-icon icon="edit_ic"></app-icon>
              </button>
            </td>
          </ng-container>
          <!-- Star Column -->
          <ng-container matColumnDef="star" stickyEnd>
            <th
              class="hidden"
              mat-header-cell
              *matHeaderCellDef
              aria-label="row actions"
            ></th>
            <td class="hidden" mat-cell *matCellDef="let element"></td>
          </ng-container>
          <!-- </ng-container> -->
          <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
          <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
        </table>
      </section>
    </div>
    <!-- <app-pagination
     *ngIf="hiringList.length > 0"
      [length]="totalRecords"
      [pageSize]="query.limit"
      (onPageChange)="onPageChange($event)"
    ></app-pagination> -->
  </div>
  <!-- <div class="mt-3">
    <div class="alert alert-primary text-center" *ngIf="hiringList.length === 0">No Application to Show</div>
  </div> -->
</div>

data format

[
    {
        "id": "de6203ee-482a-4640-95b5-25c3d09edb2b",
        "createdDate": "2024-11-26T15:17:12.402+00:00",
        "updatedDate": "2024-11-27T09:19:44.474+00:00",
        "active": true,
        "createdBy": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "updatedBy": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "job": {
            "id": "7f5f82bc-d604-4072-acf2-f1b88b1ce8ba",
            "jobTitle": "Voice - Experience - IV - Jaipur",
            "oracleJobReference": "447",
            "lineOfBusiness": "ORA_PROFESSIONAL",
            "location": "Jaipur",
            "locationId": "300000012250436",
            "shift": "NIGHT_SHIFT",
            "shortDesc": "Minimum 1 - 5 Years of Experience in Denial Management, Immediate Joiners Preferred",
            "longDesc": "<p><strong>ROLE SUMMARY</strong></p>\n<p>&nbsp;</p>\n<p>This position is based out of the Philippines- Manila office, responsible for the following:&nbsp;</p>\n<p>&nbsp;</p>\n<ul>\n <li>&nbsp;Ensure quality driven follow-up activities and resolution of accounts is carried out with the insurance carriers on the outstanding inventory to yield maximum cash flow and minimum bad debts</li>\n <li>Interact by Phone and Internet based portals with the insurance companies in US to procure status of the claim followed by appropriate activities to address open AR</li>\n <li>The key functions of an AR Executive are to ensure the below responsibilities are carried out to the best of his/her ability in the interest of the organization and client</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>RESPONSIBILITIES</strong></p>\n<p>&nbsp;</p>\n<ul>\n <li>To address outstanding or assigned AR through analysis and phone calls by using available resources</li>\n <li>Utilization of all possible tools and applications available to take account to the next level of resolution, which would result in a payment, corrected submission, appeals, patient transfer or adjustment</li>\n <li>To report trends / patterns in denials, claim submission errors, credentialing issues and billing related roadblocks to the immediate reporting manager.</li>\n <li>To meet the established SLAs (service level agreements) for production and quality</li>\n <li>To update the outcome of the calls or analysis in a clear and coherent manner in the billing system</li>\n <li>To utilize the P &amp; P’s (policies and procedures) established for the process and stay updated with changes done with the P &amp; Ps</li>\n <li>To improve the performance based on the feedback provided by the reporting manager / quality audit team</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>COMPLIANCE</strong></p>\n<p>Awareness and adherence to all&nbsp;applicable organization wide policies and procedures, including but not limited to Information security, HIPAA, and HR policies.</p>\n<p>&nbsp;</p>\n<p><strong>ACADEMIC AND PROFESSIONAL BACKGROUND</strong></p>\n<ul>\n <li>University graduate with an average aptitude score</li>\n <li>Minimum 12 months – 4 years of AR follow-up experience with either physician or hospital billing background</li>\n <li>CPAT/CCAT certification will be an added advantage</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>COMPETENCIES, SKILLS, AND OTHER REQUISITES</strong></p>\n<p>&nbsp;</p>\n<ul>\n <li>Sound analytical skills</li>\n <li>Logical thinking</li>\n <li>Attention to detail</li>\n <li>Adequate domain knowledge on various billing terminologies &amp; processes</li>\n <li>Good verbal and written communication skills</li>\n</ul>\n<p>&nbsp;</p>\n<p><strong>COMPENSATION AND BENEFITS -&nbsp;</strong>Competitive remuneration + annual performance-based bonus + standard industry benefits. Relocation package, per company policy will apply for candidates outside the office base city.&nbsp;</p>\n<p>&nbsp;</p>\n<p><strong>ASSESSMENT / INTERVIEW PROCESS</strong></p>\n<p>&nbsp;</p>\n<ol>\n <li>HR Interview</li>\n <li>Domain assessment&nbsp;</li>\n <li>Final Interview (Domain interview by Op’s)&nbsp;</li>\n</ol>",
            "openPositions": 100,
            "postedAt": "2023-01-18T06:22:08.455+00:00",
            "visibility": true,
            "jobType": "CONTRACT",
            "processType": "VOICE",
            "jobHours": "FULL_TIME",
            "experienceLevel": "Experienced",
            "cashBonus": 0,
            "education": null,
            "oracleJobReferenceNumber": "300000129588520",
            "jobImage": null,
            "jobImageUrl": null,
            "lob": "IV",
            "blockApplyButton": false,
            "error": null,
            "coolingPeriod": false,
            "currencyType": null,
            "deleted": false
        },
        "requestType": "New Position/Additional Job Role",
        "requisitionId": "2066",
        "clientName": "AGS",
        "openNumbers": 1,
        "intentDate": "2024-10-25T00:00:00.000+00:00",
        "closureDate": "2024-11-25T00:00:00.000+00:00",
        "tat": 31,
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
        "spoc": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "split": "NA",
        "role": "Support",
        "onboardingDate": null,
        "employeeReferral": 0,
        "direct": 0,
        "vendor": 0,
        "campus": 0,
        "male": 0,
        "female": 0,
        "manager": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "published": true,
        "remarks": null,
        "monthToBeOnboarded": null
    }
]



