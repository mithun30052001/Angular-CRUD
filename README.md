https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-filter.component.ts

import {
  REQUEST_TYPE_OPTIONS,
  PROCESS_OPTIONS,
  SPLIT_OPTIONS,
  ROLE_OPTIONS,
  SPECIALITY_OPTIONS,
  JOB_TYPE_OPTIONS,
  LOCATION_OPTIONS,
  SHIFT_OPTIONS,
} from '@/src/app/constants/app.constants';
import { Filters } from '@/src/app/interfaces/app-interface';
import { SharedService } from '@/src/app/subject-module/shared.service';
import { Component, Inject, isDevMode } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialogRef } from '@angular/material/dialog';
import { MatRadioChange } from '@angular/material/radio';
import { checkHrOrAdmin } from '../../utils/utils';
import { Router } from '@angular/router';

@Component({
  selector: 'app-req-filter-modal',
  templateUrl: './req-filter-modal.component.html',
  styleUrls: ['./req-filter-modal.component.scss'],
})
export class ReqFilterModalComponent {
  get REQUEST_TYPE_OPTIONS() {
    return REQUEST_TYPE_OPTIONS;
  }
  get PROCESS_OPTIONS() {
    return PROCESS_OPTIONS;
  }
  get SPLIT_OPTIONS() {
    return SPLIT_OPTIONS;
  }
  get ROLE_OPTIONS() {
    return ROLE_OPTIONS;
  }
  get SPECIALITY_OPTIONS() {
    return SPECIALITY_OPTIONS;
  }
  get JOB_TYPE_OPTIONS() {
    return JOB_TYPE_OPTIONS;
  }
  get LOCATION_OPTIONS() {
    return LOCATION_OPTIONS;
  }
  get SHIFT_OPTIONS() {
    return SHIFT_OPTIONS;
  }

  selectedFilters: any = {};
  constructor(
    private dialogRef: MatDialogRef<ReqFilterModalComponent>,
    private subjectService: SharedService,
    private router: Router,
    @Inject(MAT_DIALOG_DATA)
    public data: {
      type: 'jobs' | 'referrals' | 'applications';
      filters: any;
    }
  ) {
    this.selectedFilters = data.filters;
  }

  closeModal() {
    this.dialogRef.close();
  }

  clearFiler() {
    this.selectedFilters = {
      request_type: undefined,
      process: undefined,
      split: undefined,
      role: undefined,
      speciality: undefined,
      job_type: undefined,
      location: undefined,
      shift: undefined,
    };
    this.subjectService.setFilters({});
    this.dialogRef.close();
  }

  onClearFilter(field: any) {
    this.selectedFilters[field] = undefined;
    this.subjectService.setFilters(this.selectedFilters);
  }

  clearFilter() {
    this.selectedFilters = {};
    this.dialogRef.close();
  }

  applyFilter() {
    // Encode filters as Base64 and add to query params
    const filterString = JSON.stringify(this.selectedFilters);
    const base64Filters = btoa(filterString);
    console.log("BASE 64", base64Filters);

    this.router.navigate([], {
      queryParams: { filters: base64Filters },
      queryParamsHandling: 'merge',
    });
    this.dialogRef.close(this.selectedFilters);
  }

  onFilterSelect(value: string, field: string) {
    if (!this.selectedFilters[field]) {
      this.selectedFilters[field] = [];
    }

    if (this.selectedFilters[field].includes(value)) {
      this.selectedFilters[field] = this.selectedFilters[field].filter(
        (item: string) => item !== value
      );
    } else {
      this.selectedFilters[field].push(value);
    }

    if (isDevMode()) {
      console.log('Selected filters:', this.selectedFilters);
    }
  }
}


req-filter.component.html
<!-- Job Card Filter -->

<div class="filter-view">
  <div class="heading">
    <div class="heading-text">Filter</div>
    <div class="heading-image">
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>
    </div>
  </div>
  <div class="filter-overflow">
    <!-- Request Type -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Request Type</div>
          <button
            *ngIf="selectedFilters.request_type?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('request_type')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div
              class="col-sm-3"
              *ngFor="let request_type of REQUEST_TYPE_OPTIONS"
            >
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="
                      selectedFilters.request_type?.includes(request_type.value)
                    "
                    (change)="
                      onFilterSelect(request_type.value, 'request_type')
                    "
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ request_type.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Process -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Process</div>
          <button
            *ngIf="selectedFilters.process?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('process')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let process of PROCESS_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="selectedFilters.process?.includes(process.value)"
                    (change)="onFilterSelect(process.value, 'process')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ process.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Split -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Split</div>
          <button
            *ngIf="selectedFilters.split?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('split')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let split of SPLIT_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="selectedFilters.split?.includes(split.value)"
                    (change)="onFilterSelect(split.value, 'split')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ split.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Role -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Role</div>
          <button
            *ngIf="selectedFilters.role?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('role')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let role of ROLE_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="selectedFilters.role?.includes(role.value)"
                    (change)="onFilterSelect(role.value, 'role')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ role.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Speciality -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Speciality</div>
          <button
            *ngIf="selectedFilters.speciality?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('speciality')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let speciality of SPECIALITY_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="
                      selectedFilters.process?.includes(speciality.value)
                    "
                    (change)="onFilterSelect(speciality.value, 'speciality')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ speciality.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Job Type -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Job Type</div>
          <button
            *ngIf="selectedFilters.job_type?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('job_type')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let job_type of JOB_TYPE_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="
                      selectedFilters.process?.includes(job_type.value)
                    "
                    (change)="onFilterSelect(job_type.value, 'job_type')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ job_type.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Job Location -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Location</div>
          <button
            *ngIf="selectedFilters.location?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('location')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let location of LOCATION_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="
                      selectedFilters.location?.includes(location.value)
                    "
                    (change)="onFilterSelect(location.value, 'location')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ location.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Shift -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-sm-2">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Shift</div>
          <button
            *ngIf="selectedFilters.shift?.length"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('shift')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-sm-10">
        <div class="right-panel">
          <div class="row">
            <div class="col-sm-3" *ngFor="let shift of SHIFT_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-checkbox
                    class="mt-0 mb-1"
                    [checked]="selectedFilters.shift?.includes(shift.value)"
                    (change)="onFilterSelect(shift.value, 'shift')"
                  ></mat-checkbox>
                </div>
                <div class="right-panel-location">
                  {{ shift.display }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="footer-button">
    <div class="card-style">
      <!-- <div>
        <button
          title="Clear Filter"
          class="ags-outline-btn ags-hxl56 btn-font16 ags-padding1624"
          (click)="clearFiler()"
        >
          Clear Filter
        </button>
      </div> -->
      <div>
        <button
          title="Apply Filter"
          class="ags-primary-btn ags-hxl56 btn-font16 ags-padding1624"
          (click)="applyFilter()"
        >
          Apply Filter
        </button>
      </div>
    </div>
  </div>
</div>

