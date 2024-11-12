side-nav.component.ts

import { ResponsiveViewComponent } from '@/shared/components/responsive-view/responsive-view.component';
import {
  EXPERIENCE_LEVEL_OPTIONS,
  CANDIDATE_OPTIONS,
  SOURCE_OPTIONS,
  LOCATION_OPTIONS,
  PROCESS_TYPE_OPTIONS,
  SHIFT_OPTIONS,
  STATUS_OPTIONS,
  ACCOUNT_STATUS_OPTIONS,
  TASK_STATUS_OPTIONS,
  HR_STATUS_OPTIONS,
} from '@/src/app/constants/app.constants';
import { Filters } from '@/src/app/interfaces/app-interface';
import {
  checkCandidateAccess,
  checkEmployeeAccess,
  checkHrOrAdmin,
  checkVendorAccess,
} from '@/src/app/shared/utils/utils';
import { SharedService } from '@/src/app/subject-module/shared.service';
import { MediaMatcher } from '@angular/cdk/layout';
import { Component, Input, OnDestroy, OnInit, isDevMode } from '@angular/core';
import { MatCheckboxChange } from '@angular/material/checkbox';
import { MatRadioChange } from '@angular/material/radio';
import { Router, ActivatedRoute } from '@angular/router';
import { faLessThan } from '@fortawesome/free-solid-svg-icons';

@Component({
  selector: 'app-side-nav',
  templateUrl: './side-nav.component.html',
  styleUrls: ['./side-nav.component.scss'],
})
export class SideNavComponent
  extends ResponsiveViewComponent
  implements OnInit, OnDestroy
{
  // readonly panelOpenState = signal(false);

  panelOpenState: boolean[] = [false, false];
  @Input() mode: 'side' | 'bottom' = 'side';

  checkCandidateAccess = checkCandidateAccess;
  checkEmployeeAccess = checkEmployeeAccess;
  // checkHrAccess = checkHrAccess;
  checkHrOrAdmin = checkHrOrAdmin;
  checkVendorAccess = checkVendorAccess;
  pathName = '';
  completeMenu = true;
  statusMenu = false;
  userProfile = 'assets/images/userProfile.png';
  jobListing = 'assets/images/job-listing.svg';
  applicationStatus = 'assets/images/application-status.svg';
  referral = 'assets/images/referrals.svg';
  openPositions = 'assets/images/job-listing-employee.svg';
  referrals = 'assets/images/referrals.svg';
  hrCandidateList = 'assets/images/userLight.svg';
  hrJobList = 'assets/images/job-listing.svg';

  selectedStatusIndex = undefined;
  selectedLocationIndex = undefined;
  selectedJobTimingIndex = undefined;
  selectedJobTypeIndex = undefined;
  selectedExperienceIndex = undefined;

  selectedFilters: Filters = {};
  user_type: string | undefined = '';
  demo: any;
  filterSelected: any[] = [];
  finalFilter: any[] = [];
  statusFilter: any[] = [];
  statusFilterArray: any[] = [];
  statusQueryFilter: any[] = [];
  locationQueryFilterArray: any[] = [];
  locationQueryFilter: any[] = [];
  shiftQueryFilter: any[] = [];
  shiftQueryFilterArray: any[] = [];
  lineOfBusinessQueryFilter: any[] = [];
  lineOfBusinessQueryFilterArray: any[] = [];
  experienceQueryFilter: any[] = [];
  experienceQueryFilterArray: any[] = [];
  filterCountNumber = 0;
  isVendorMenu = false;
  hrVendorMenu = false;
  hrCandidateListMenu = false;

  get CANDIDATE_OPTION() {
    return CANDIDATE_OPTIONS;
  }
  get SOURCE_OPTIONS() {
    return SOURCE_OPTIONS;
  }

  get LOCATION_OPTIONS() {
    return LOCATION_OPTIONS;
  }
  get SHIFT_OPTIONS() {
    return SHIFT_OPTIONS;
  }
  get PROCESS_TYPE_OPTIONS() {
    return PROCESS_TYPE_OPTIONS;
  }
  get EXPERIENCE_LEVEL_OPTIONS() {
    return EXPERIENCE_LEVEL_OPTIONS;
  }

  get STATUS_OPTIONS() {
    if (checkHrOrAdmin()) return HR_STATUS_OPTIONS;
    else return STATUS_OPTIONS;
  }

  get ACCOUNT_STATUS_OPTIONS() {
    return ACCOUNT_STATUS_OPTIONS;
  }

  get TASK_STATUS_OPTIONS() {
    return TASK_STATUS_OPTIONS;
  }

  isPanelMember = false;
  constructor(
    media: MediaMatcher,
    private subjectService: SharedService,
    private router: Router,
    private activatedRoute: ActivatedRoute
  ) {
    super(media);
    this.isPanelMember = Boolean(localStorage.getItem('panel'));
    this.pathName = window.location.pathname;
    this.setUserType();
    if (
      this.pathName.includes('c/jobs') ||
      this.pathName.includes('openings') ||
      this.pathName.includes('candidate-list')
    ) {
      if (this.pathName.includes('candidate-list')) {
        this.hrCandidateListMenu = true;
        this.completeMenu = false;
      } else {
        if (!this.pathName.includes('/candidate-profile')) {
          this.completeMenu = true;
        } else this.completeMenu = false;
      }
      this.statusMenu = false;
      this.isVendorMenu = false;
      this.hrVendorMenu = false;
    } else {
      if (this.pathName.includes('/vendor/list')) {
        this.completeMenu = false;
        this.hrVendorMenu = true;
        this.isVendorMenu = false;
        this.statusMenu = false;
      } else if (
        this.pathName.includes('application') ||
        this.pathName.includes('employee-referrals') ||
        this.pathName.includes('referral')
      ) {
        this.completeMenu = false;
        this.statusMenu = true;
        this.isVendorMenu = false;
        this.hrVendorMenu = false;
      } else if (
        this.pathName.includes('v/vendor-dashboard') ||
        this.pathName.includes('v/candidates')
      ) {
        console.log('in vendor ');
        this.completeMenu = false;
        this.statusMenu = false;
        this.isVendorMenu = true;
        this.hrVendorMenu = false;
      } else {
        this.completeMenu = false;
        this.statusMenu = false;
        this.isVendorMenu = false;
        this.hrVendorMenu = false;
      }
    }
  }

  setUserType() {
    this.user_type = localStorage
      .getItem('role')
      ?.toString()
      .toLocaleLowerCase()
      .trim();
  }

  cccc(event: boolean, page: string) {
    console.log(event, page);
  }

  ngOnInit(): void {
    this.setUserType();
    this.userProfile = 'assets/images/userProfile-clicked.svg';
    this.jobListing = 'assets/images/job-listing.svg';
    this.applicationStatus = 'assets/images/application-status.svg';
    this.referral = 'assets/images/referrals.svg';
    this.openPositions = 'assets/images/job-listing-employee-clicked.svg';
    this.referrals = 'assets/images/referrals.svg';
    this.getMenuRole();
  }

  ngOnDestroy(): void {
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.subjectService.setFilters({});
  }
  getMenuRole() {
    return localStorage.getItem('role');
  }

  candidateHrMenuSet() {
    this.subjectService.setFilters({});
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.userProfile = 'assets/images/userProfile.png';
    this.jobListing = 'assets/images/job-listing-clicked.png';
    this.applicationStatus = 'assets/images/application-status.svg';
    this.referral = 'assets/images/referrals.svg';
    this.openPositions = 'assets/images/job-listing-employee-clicked.svg';
    this.referrals = 'assets/images/referrals.svg';
    this.completeMenu = false;
    this.statusMenu = false;
    this.hrVendorMenu = false;
    this.isVendorMenu = false;
    this.hrCandidateListMenu = true;
  }

  vendorMenuSet() {
    this.subjectService.setFilters({});
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.userProfile = 'assets/images/userProfile.png';
    this.jobListing = 'assets/images/job-listing-clicked.png';
    this.applicationStatus = 'assets/images/application-status.svg';
    this.referral = 'assets/images/referrals.svg';
    this.openPositions = 'assets/images/job-listing-employee-clicked.svg';
    this.referrals = 'assets/images/referrals.svg';
    this.completeMenu = false;
    this.statusMenu = false;
    this.hrVendorMenu = false;
    this.isVendorMenu = true;
    this.hrCandidateListMenu = false;
  }

  completeMenuSet() {
    this.subjectService.setFilters({});
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.userProfile = 'assets/images/userProfile.png';
    this.jobListing = 'assets/images/job-listing-clicked.png';
    this.applicationStatus = 'assets/images/application-status.svg';
    this.referral = 'assets/images/referrals.svg';
    this.openPositions = 'assets/images/job-listing-employee-clicked.svg';
    this.referrals = 'assets/images/referrals.svg';
    this.completeMenu = true;
    this.statusMenu = false;
    this.hrVendorMenu = false;
    this.isVendorMenu = false;
    this.hrCandidateListMenu = false;
  }

  statusMenuSet() {
    this.subjectService.setFilters({});
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.userProfile = 'assets/images/userProfile.png';
    this.jobListing = 'assets/images/job-listing.svg';
    this.applicationStatus = 'assets/images/application-status-clicked.png';
    this.referral = 'assets/images/referrals.svg';
    this.openPositions = 'assets/images/job-listing-employee.svg';
    this.referrals = 'assets/images/referrals-clicked.png';
    this.completeMenu = false;
    this.statusMenu = true;
    this.hrVendorMenu = false;
    this.isVendorMenu = false;
    this.hrCandidateListMenu = false;
  }

  removeMenuSet() {
    this.subjectService.setFilters({});
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.userProfile = 'assets/images/userProfile-clicked.svg';
    this.jobListing = 'assets/images/job-listing.svg';
    this.applicationStatus = 'assets/images/application-status.svg';
    this.referral = 'assets/images/referrals.svg';
    this.completeMenu = false;
    this.statusMenu = false;
    this.hrVendorMenu = false;
    this.isVendorMenu = false;
    this.hrCandidateListMenu = false;
  }
  referralStatusMenuSet() {
    this.subjectService.setFilters({});
    this.selectedFilters = {
      account: undefined,
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
      source: undefined,
      candidateType: undefined,
    };
    this.userProfile = 'assets/images/userProfile.png';
    this.jobListing = 'assets/images/job-listing.svg';
    this.applicationStatus = 'assets/images/application-status.svg';
    this.referral = 'assets/images/referrals-clicked.png';
    this.completeMenu = false;
    this.statusMenu = true;
    this.hrVendorMenu = false;
    this.isVendorMenu = false;
    this.hrCandidateListMenu = false;
  }

  onClearFilter(field: keyof Filters) {
    this.selectedFilters[field] = undefined;
    this.subjectService.setFilters(this.selectedFilters);
  }

  onFilterSelect(event: MatRadioChange, field: keyof Filters) {
    this.selectedFilters = {
      ...this.selectedFilters,
      [field]: event.value,
    };

    if (isDevMode()) console.log('selected filters::> ', this.selectedFilters);
    this.subjectService.setFilters(this.selectedFilters);
  }

  onCheckFilterSelect(event: MatCheckboxChange, field: keyof Filters) {
    if (this.selectedFilters.status) {
      var arr = this.selectedFilters.status.split(',');
      if (event.checked) arr.push(event.source.value);
      else arr = arr.filter((x: string) => x != event.source.value);

      this.selectedFilters.status = arr.length > 0 ? arr.join(',') : null;
    } else {
      this.selectedFilters.status = event.source.value;
    }

    if (this.selectedFilters)
      this.selectedFilters = {
        ...this.selectedFilters,
        [field]: this.selectedFilters.status,
      };

    if (isDevMode()) console.log('selected filters::> ', this.selectedFilters);
    this.subjectService.setFilters(this.selectedFilters);
  }

  isActive(url: string): boolean {
    return this.router.isActive(url, {
      paths: 'subset',
      queryParams: 'ignored',
      fragment: 'ignored',
      matrixParams: 'ignored',
    });
  }
  isButtonActive(): boolean {
    const currentParam = this.activatedRoute.snapshot.queryParams['v'];

    return (
      this.isActive('h/calendar/view') &&
      (currentParam === 'week' ||
        currentParam === 'month' ||
        currentParam === 'day')
    );
  }
  change($event: any) {
    console.log($event);
  }
}


side-nav.component.html
<div *ngIf="!mobileView && mode === 'side'">
  <div *ngIf="user_type === 'candidate'; then candidateMenu"></div>
  <div *ngIf="user_type === 'employee_non_hr'; then employeeMenu"></div>
  <div
    *ngIf="
      user_type === 'employee_hr' || user_type === 'employee_hr_admin';
      then checkHrMenu
    "
  ></div>
  <div *ngIf="user_type === 'vendor'; then vendorMenu"></div>
  <ng-template #candidateMenu>
    <div class="menu-list">
      <div class="menu-item">
        <button
          title="Profile"
          mat-button
          color="primary"
          routerLink="c/candidate-profile"
          [routerLinkActive]="'activeButton'"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('c/candidate-profile') ? 'profile_active' : 'profile'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Profile</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Job Listing"
          mat-button
          color="primary"
          routerLink="c/jobs"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('c/jobs') ? 'job_listing_active' : 'job_listing'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Job Listing</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Application Status"
          mat-button
          color="primary"
          routerLink="c/application-status"
          [routerLinkActive]="'activeButton'"
          (click)="statusMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('c/application-status')
                    ? 'application_status_active'
                    : 'application_status'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Application Status</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Referrals"
          mat-button
          color="primary"
          routerLink="c/referral-status"
          [routerLinkActive]="'activeButton'"
          (click)="referralStatusMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('c/referral-status')
                    ? 'referrals_active'
                    : 'referrals'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Referrals</div>
          </div>
        </button>
      </div>
    </div>
  </ng-template>
  <ng-template #employeeMenu>
    <div class="menu-list" completeMenu>
      <div class="menu-item">
        <button
          title="Open Positions"
          mat-button
          color="primary"
          routerLink="e/openings"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('e/openings') ? 'job_listing_active' : 'job_listing'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Open Positions</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="My Referrals"
          mat-button
          color="primary"
          routerLink="e/employee-referrals"
          [routerLinkActive]="'activeButton'"
          (click)="statusMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('e/employee-referrals')
                    ? 'referrals_active'
                    : 'referrals'
                "
              ></app-icon>
            </div>
            <div class="menu-text">My Referrals</div>
          </div>
        </button>
      </div>
      <div *ngIf="isPanelMember" class="menu-item">
        <button
          title="Interviews"
          mat-button
          color="primary"
          routerLink="e/interviews"
          [routerLinkActive]="'activeButton'"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('e/interviews') ? 'interviews_active' : 'interviews'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Interviews</div>
          </div>
        </button>
      </div>
      <div *ngIf="isPanelMember" class="menu-item">
        <button
          title="Calendar"
          mat-button
          color="primary"
          routerLink="e/calendar"
          [routerLinkActive]="'activeButton'"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="isActive('e/calendar') ? 'calendar_active' : 'calendar'"
              ></app-icon>
            </div>
            <div class="menu-text">Calendar</div>
          </div>
        </button>
      </div>
    </div>
  </ng-template>
  <ng-template #checkHrMenu>
    <div class="menu-list">
      <div class="menu-item">
        <button
          title="Candidate List"
          mat-button
          color="primary"
          routerLink="h/candidate-list"
          [routerLinkActive]="'activeButton'"
          (click)="candidateHrMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('h/candidate-list') ? 'profile_active' : 'profile'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Candidate List</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Job Listing"
          mat-button
          color="primary"
          routerLink="h/openings"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('h/openings') ? 'job_listing_active' : 'job_listing'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Job Listing</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Vendors"
          mat-button
          color="primary"
          routerLink="h/vendor"
          [routerLinkActive]="'activeButton'"
          (click)="vendorMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="isActive('h/vendor') ? 'vendor_active' : 'vendor'"
              ></app-icon>
            </div>
            <div class="menu-text">Vendors</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Calendar"
          mat-button
          color="primary"
          routerLink="h/calendar/view"
          [routerLinkActive]="'activeButton'"
          #rla="routerLinkActive"
          (isActiveChange)="change($event)"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="rla.isActive ? 'calendar_active' : 'calendar'"
              ></app-icon>
            </div>
            <div class="menu-text">Calendar</div>
          </div>
        </button>
      </div>
    </div>
  </ng-template>
  <ng-template #vendorMenu>
    <div class="menu-list">
      <div class="menu-item">
        <button
          title="Dashboard"
          mat-button
          color="primary"
          routerLink="v/vendor-dashboard"
          [routerLinkActive]="'activeButton'"
          (click)="vendorMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('v/vendor-dashboard')
                    ? 'dashboard_active'
                    : 'dashboard'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Dashboard</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Candidate Status"
          mat-button
          color="primary"
          routerLink="v/candidates"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('v/candidates')
                    ? 'candidate_status_active'
                    : 'candidate_status'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Candidate Status</div>
          </div>
        </button>
      </div>
      <div class="menu-item">
        <button
          title="Profile"
          mat-button
          color="primary"
          routerLink="v/profile"
          [routerLinkActive]="'activeButton'"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="
                  isActive('v/profile')
                    ? 'vendor_profile_active'
                    : 'vendor_profile'
                "
              ></app-icon>
            </div>
            <div class="menu-text">Profile</div>
          </div>
        </button>
      </div>
    </div>
  </ng-template>

  <div *ngIf="completeMenu" class="below-menu">
    <mat-accordion multi>
      <mat-expansion-panel
        (opened)="panelOpenState[0] = true"
        (closed)="panelOpenState[0] = false"
        hideToggle="true"
      >
        <mat-expansion-panel-header>
          <div class="">
            <div class="d-flex justify-content-between align-items-center">
              <div class="job-location-heading">Job Location</div>
              <app-icon icon="add" *ngIf="!panelOpenState[0]"></app-icon>
              <app-icon icon="remove" *ngIf="panelOpenState[0]"></app-icon>
            </div>
          </div>
        </mat-expansion-panel-header>
        <button
          *ngIf="selectedFilters.location"
          title="clear filters"
          class="clear-btn"
          aria-label="clear filters"
          (click)="onClearFilter('location'); $event.stopPropagation()"
        >
          Clear
        </button>
        <mat-radio-group [(ngModel)]="selectedFilters.location">
          <div *ngFor="let location of LOCATION_OPTIONS; let index = index">
            <div class="items">
              <div class="item-checkbox">
                <mat-radio-button
                  [value]="location.value"
                  (change)="onFilterSelect($event, 'location')"
                ></mat-radio-button>
              </div>
              <div class="item-text">{{ location.display }}</div>
            </div>
          </div>
        </mat-radio-group>
      </mat-expansion-panel>
    </mat-accordion>

    <div class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[1] = true"
          (closed)="panelOpenState[1] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Timing</div>
                <app-icon icon="add" *ngIf="!panelOpenState[1]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[1]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.shift"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('shift'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.shift">
            <div *ngFor="let shift of SHIFT_OPTIONS; let index = index">
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    class="example-margin"
                    [value]="shift.value"
                    (change)="onFilterSelect($event, 'shift')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ shift.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="type-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[2] = true"
          (closed)="panelOpenState[2] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Type</div>
                <app-icon icon="add" *ngIf="!panelOpenState[2]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[2]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.processType"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('processType'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.processType">
            <div
              *ngFor="
                let processType of PROCESS_TYPE_OPTIONS;
                let index = index
              "
            >
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    class="example-margin"
                    [value]="processType.value"
                    (change)="onFilterSelect($event, 'processType')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ processType.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[3] = true"
          (closed)="panelOpenState[3] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Experience Level</div>
                <app-icon icon="add" *ngIf="!panelOpenState[3]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[3]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.experienceLevel"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="
                  onClearFilter('experienceLevel'); $event.stopPropagation()
                "
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.experienceLevel">
            <div
              *ngFor="
                let experienceLevel of EXPERIENCE_LEVEL_OPTIONS;
                let index = index
              "
            >
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    [value]="experienceLevel.value"
                    class="example-margin"
                    (change)="onFilterSelect($event, 'experienceLevel')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ experienceLevel.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>
  </div>

  <div *ngIf="hrCandidateListMenu" class="below-menu">
    <div class="filter-section">
      <!-- <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[8] = true"
          (closed)="panelOpenState[8] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header> -->
      <div class="">
        <div class="d-flex justify-content-between align-items-center">
          <!-- <div class="job-location-heading">Candidate Type</div> -->
          <!-- <app-icon icon="add" *ngIf="!panelOpenState[8]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[8]"></app-icon> -->
        </div>
        <button
          *ngIf="selectedFilters.candidateType"
          title="clear filters"
          class="clear-btn"
          aria-label="clear filters"
          (click)="onClearFilter('candidateType'); $event.stopPropagation()"
        >
          Clear
        </button>
      </div>
      <!-- </mat-expansion-panel-header> -->
      <mat-radio-group [(ngModel)]="selectedFilters.candidateType">
        <div *ngFor="let candidate of CANDIDATE_OPTION; let index = index">
          <div class="items">
            <div class="item-checkbox">
              <mat-radio-button
                [value]="candidate.value"
                class="example-margin"
                (change)="onFilterSelect($event, 'candidateType')"
              ></mat-radio-button>
            </div>
            <div class="item-text">{{ candidate.display }}</div>
          </div>
        </div>
      </mat-radio-group>
      <!-- </mat-expansion-panel> -->
      <!-- </mat-accordion> -->
    </div>

    <mat-accordion multi>
      <mat-expansion-panel
        (opened)="panelOpenState[0] = true"
        (closed)="panelOpenState[0] = false"
        hideToggle="true"
      >
        <mat-expansion-panel-header>
          <div class="">
            <div class="d-flex justify-content-between align-items-center">
              <div class="job-location-heading">Job Location</div>
              <app-icon icon="add" *ngIf="!panelOpenState[0]"></app-icon>
              <app-icon icon="remove" *ngIf="panelOpenState[0]"></app-icon>
            </div>
            <button
              *ngIf="selectedFilters.location"
              title="clear filters"
              class="clear-btn"
              aria-label="clear filters"
              (click)="onClearFilter('location'); $event.stopPropagation()"
            >
              Clear
            </button>
          </div>
        </mat-expansion-panel-header>
        <mat-radio-group [(ngModel)]="selectedFilters.location">
          <div *ngFor="let location of LOCATION_OPTIONS; let index = index">
            <div class="items">
              <div class="item-checkbox">
                <mat-radio-button
                  [value]="location.value"
                  (change)="onFilterSelect($event, 'location')"
                ></mat-radio-button>
              </div>
              <div class="item-text">{{ location.display }}</div>
            </div>
          </div>
        </mat-radio-group>
      </mat-expansion-panel>
    </mat-accordion>

    <div *ngIf="!checkHrOrAdmin()" class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[1] = true"
          (closed)="panelOpenState[1] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Timing</div>
                <app-icon icon="add" *ngIf="!panelOpenState[1]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[1]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.shift"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('shift'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.shift">
            <div *ngFor="let shift of SHIFT_OPTIONS; let index = index">
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    class="example-margin"
                    [value]="shift.value"
                    (change)="onFilterSelect($event, 'shift')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ shift.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="type-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[2] = true"
          (closed)="panelOpenState[2] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Type</div>
                <app-icon icon="add" *ngIf="!panelOpenState[2]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[2]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.processType"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('processType'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.processType">
            <div
              *ngFor="
                let processType of PROCESS_TYPE_OPTIONS;
                let index = index
              "
            >
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    class="example-margin"
                    [value]="processType.value"
                    (change)="onFilterSelect($event, 'processType')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ processType.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[3] = true"
          (closed)="panelOpenState[3] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Experience Level</div>
                <app-icon icon="add" *ngIf="!panelOpenState[3]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[3]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.experienceLevel"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="
                  onClearFilter('experienceLevel'); $event.stopPropagation()
                "
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.experienceLevel">
            <div
              *ngFor="
                let experienceLevel of EXPERIENCE_LEVEL_OPTIONS;
                let index = index
              "
            >
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    [value]="experienceLevel.value"
                    class="example-margin"
                    (change)="onFilterSelect($event, 'experienceLevel')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ experienceLevel.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[4] = true"
          (closed)="panelOpenState[4] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Source</div>
                <app-icon icon="add" *ngIf="!panelOpenState[4]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[4]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.source"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('source'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.source">
            <div *ngFor="let source of SOURCE_OPTIONS; let index = index">
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    [value]="source.value"
                    class="example-margin"
                    (change)="onFilterSelect($event, 'source')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ source.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[9] = true"
          (closed)="panelOpenState[9] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Status</div>
                <app-icon icon="add" *ngIf="!panelOpenState[9]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[9]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.status"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('status'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <div *ngFor="let status of STATUS_OPTIONS; let index = index">
            <div class="items">
              <div class="item-checkbox">
                <mat-checkbox
                  [checked]="selectedFilters.status?.includes(status.value)"
                  [value]="status.value"
                  class="example-margin"
                  (change)="onCheckFilterSelect($event, 'status')"
                ></mat-checkbox>
              </div>
              <div class="item-text">{{ status.display }}</div>
            </div>
          </div>
        </mat-expansion-panel>
      </mat-accordion>
    </div>
  </div>

  <div *ngIf="statusMenu" class="below-menu">
    <div class="filter-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[5] = true"
          (closed)="panelOpenState[5] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="d-flex justify-content-between align-items-center">
              <div class="job-location-heading">Status</div>
              <app-icon icon="add" *ngIf="!panelOpenState[5]"></app-icon>
              <app-icon icon="remove" *ngIf="panelOpenState[5]"></app-icon>
            </div>
            <button
              *ngIf="selectedFilters.status"
              title="clear filters"
              class="clear-btn"
              aria-label="clear filters"
              (click)="onClearFilter('status'); $event.stopPropagation()"
            >
              Clear
            </button>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.status">
            <div *ngFor="let status of STATUS_OPTIONS; let index = index">
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    [value]="status.value"
                    class="example-margin"
                    (change)="onFilterSelect($event, 'status')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ status.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>
  </div>

  <div *ngIf="isVendorMenu" class="below-menu">
    <div class="job-location">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[6] = true"
          (closed)="panelOpenState[6] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Job Location</div>
                <app-icon icon="add" *ngIf="!panelOpenState[6]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[6]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.location"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('location'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.location">
            <div *ngFor="let location of LOCATION_OPTIONS; let index = index">
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    [value]="location.value"
                    (change)="onFilterSelect($event, 'location')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ location.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="type-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[2] = true"
          (closed)="panelOpenState[2] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Type</div>
                <app-icon icon="add" *ngIf="!panelOpenState[2]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[2]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.processType"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('processType'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.processType">
            <div
              *ngFor="
                let processType of PROCESS_TYPE_OPTIONS;
                let index = index
              "
            >
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    class="example-margin"
                    [value]="processType.value"
                    (change)="onFilterSelect($event, 'processType')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ processType.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>

    <div class="type-section">
      <mat-accordion multi>
        <mat-expansion-panel
          (opened)="panelOpenState[7] = true"
          (closed)="panelOpenState[7] = false"
          hideToggle="true"
        >
          <mat-expansion-panel-header>
            <div class="">
              <div class="d-flex justify-content-between align-items-center">
                <div class="job-location-heading">Task status</div>
                <app-icon icon="add" *ngIf="!panelOpenState[7]"></app-icon>
                <app-icon icon="remove" *ngIf="panelOpenState[7]"></app-icon>
              </div>
              <button
                *ngIf="selectedFilters.active"
                title="clear filters"
                class="clear-btn"
                aria-label="clear filters"
                (click)="onClearFilter('active'); $event.stopPropagation()"
              >
                Clear
              </button>
            </div>
          </mat-expansion-panel-header>
          <mat-radio-group [(ngModel)]="selectedFilters.active">
            <div *ngFor="let active of TASK_STATUS_OPTIONS; let index = index">
              <div class="items">
                <div class="item-checkbox">
                  <mat-radio-button
                    class="example-margin"
                    [value]="active.value"
                    (change)="onFilterSelect($event, 'active')"
                  ></mat-radio-button>
                </div>
                <div class="item-text">{{ active.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </mat-expansion-panel>
      </mat-accordion>
    </div>
  </div>
</div>

<div *ngIf="mobileView && mode === 'bottom'">
  <div *ngIf="user_type === 'candidate'; then candidateMenu"></div>
  <div *ngIf="user_type === 'employee_non_hr'; then employeeMenu"></div>
  <div
    *ngIf="
      user_type === 'employee_hr' || user_type === 'employee_hr_admin';
      then hrMenu
    "
  ></div>

  <ng-template #candidateMenu>
    <div class="menu-list-bottom">
      <div class="menu-item-bottom">
        <button
          title="Profile"
          mat-button
          color="primary"
          routerLink="c/candidate-profile"
          [routerLinkActive]="'activeButton'"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('c/candidate-profile')
                    ? 'm_profile_active'
                    : 'm_profile'
                "
              ></app-icon>
            </div>
            <div
              *ngIf="isActive('c/candidate-profile')"
              class="menu-text-bottom"
            >
              Profile
            </div>
          </div>
        </button>
      </div>
      <div class="menu-item-bottom">
        <button
          title="Job Listing"
          mat-button
          color="primary"
          routerLink="c/jobs"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('c/jobs') ? 'm_job_listing_active' : 'm_job_listing'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('c/jobs')" class="menu-text-bottom">
              Job Listing
            </div>
          </div>
        </button>
      </div>
      <div class="menu-item-bottom">
        <button
          title="Application Status"
          mat-button
          color="primary"
          routerLink="c/application-status"
          [routerLinkActive]="'activeButton'"
          (click)="statusMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('c/application-status')
                    ? 'm_application_status_active'
                    : 'm_application_status'
                "
              ></app-icon>
            </div>
            <div
              *ngIf="isActive('c/application-status')"
              class="menu-text-bottom"
            >
              Application Status
            </div>
          </div>
        </button>
      </div>
      <div class="menu-item-bottom">
        <button
          title="Referrals"
          mat-button
          color="primary"
          routerLink="c/referral-status"
          [routerLinkActive]="'activeButton'"
          (click)="referralStatusMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('c/referral-status')
                    ? 'm_referrals_active'
                    : 'm_referrals'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('c/referral-status')" class="menu-text-bottom">
              Referrals
            </div>
          </div>
        </button>
      </div>
    </div>
  </ng-template>

  <ng-template #hrMenu>
    <div class="menu-list-bottom">
      <div class="menu-item-bottom">
        <button
          title="Candidate List"
          mat-button
          color="primary"
          routerLink="h/candidate-list"
          [routerLinkActive]="'activeButton'"
          (click)="candidateHrMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('h/candidate-list')
                    ? 'm_profile_active'
                    : 'm_profile'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('h/candidate-list')" class="menu-text-bottom">
              Candidate List
            </div>
          </div>
        </button>
      </div>
      <div class="menu-item-bottom">
        <button
          title="Job Listing"
          mat-button
          color="primary"
          routerLink="h/openings"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('h/openings')
                    ? 'm_job_listing_active'
                    : 'm_job_listing'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('h/openings')" class="menu-text-bottom">
              Job Listing
            </div>
          </div>
        </button>
      </div>
      <!-- <div class="menu-item-bottom">
        <button
          title="Vendors"
          mat-button
          color="primary"
          routerLink="h/vendor/list"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('h/vendor/list') ? 'm_vendor_active' : 'm_vendor'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('h/vendor/list')" class="menu-text-bottom">
              Vendors
            </div>
          </div>
        </button>
      </div> -->
      <div class="menu-item">
        <button
          title="Calendar"
          mat-button
          color="primary"
          routerLink="h/calendar"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid">
            <div class="menu-icon">
              <app-icon
                [icon]="isActive('h/calendar') ? 'calendar_active' : 'calendar'"
              ></app-icon>
            </div>
            <div class="menu-text">Calendar</div>
          </div>
        </button>
      </div>
    </div>
  </ng-template>

  <ng-template #employeeMenu>
    <div class="menu-list-bottom" completeMenu>
      <div class="menu-item-bottom">
        <button
          title="Open Positions"
          mat-button
          color="primary"
          routerLink="e/openings"
          [routerLinkActive]="'activeButton'"
          (click)="completeMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('e/openings')
                    ? 'm_job_listing_active'
                    : 'm_job_listing'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('e/openings')" class="menu-text-bottom">
              Open Positions
            </div>
          </div>
        </button>
      </div>
      <div class="menu-item-bottom">
        <button
          title="My Referrals"
          mat-button
          color="primary"
          routerLink="e/employee-referrals"
          [routerLinkActive]="'activeButton'"
          (click)="statusMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('e/employee-referrals')
                    ? 'm_referrals_active'
                    : 'm_referrals'
                "
              ></app-icon>
            </div>
            <div
              *ngIf="isActive('e/employee-referrals')"
              class="menu-text-bottom"
            >
              My Referrals
            </div>
          </div>
        </button>
      </div>
      <div *ngIf="isPanelMember" class="menu-item-bottom">
        <button
          title="Interviews"
          mat-button
          color="primary"
          routerLink="e/interviews"
          [routerLinkActive]="'activeButton'"
          (click)="removeMenuSet()"
        >
          <div class="menu-grid-bottom">
            <div class="menu-icon-bottom">
              <app-icon
                [icon]="
                  isActive('e/interviews')
                    ? 'm_interviews_active'
                    : 'm_interviews'
                "
              ></app-icon>
            </div>
            <div *ngIf="isActive('e/interviews')" class="menu-text-bottom">
              Interviews
            </div>
          </div>
        </button>
      </div>
    </div>
    <div *ngIf="isPanelMember" class="menu-item-bottom">
      <button
        title="Calendar"
        mat-button
        color="primary"
        routerLink="e/calendar"
        [routerLinkActive]="'activeButton'"
        (click)="removeMenuSet()"
      >
        <div class="menu-grid-bottom">
          <div class="menu-icon-bottom">
            <app-icon
              [icon]="isActive('e/calendar') ? 'calendar_active' : 'calendar'"
            ></app-icon>
          </div>
          <div *ngIf="isActive('e/calendar')" class="menu-text-bottom">
            Calendar
          </div>
        </div>
      </button>
    </div>
  </ng-template>
</div>

