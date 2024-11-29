https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

filter-view.component.ts

import {
  EXPERIENCE_LEVEL_OPTIONS,
  LOCATION_OPTIONS,
  PROCESS_TYPE_OPTIONS,
  SHIFT_OPTIONS,
  STATUS_OPTIONS,
  CANDIDATE_OPTIONS,
  SOURCE_OPTIONS,
  HR_STATUS_OPTIONS
} from '@/src/app/constants/app.constants';
import { Filters } from '@/src/app/interfaces/app-interface';
import { SharedService } from '@/src/app/subject-module/shared.service';
import { Component, Inject, isDevMode } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialogRef } from '@angular/material/dialog';
import { MatRadioChange } from '@angular/material/radio';
import { checkHrOrAdmin } from '../../utils/utils';

@Component({
  selector: 'app-filter-view-modal',
  templateUrl: './filter-view-modal.component.html',
  styleUrls: ['./filter-view-modal.component.scss'],
})
export class FilterViewModalComponent {
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
  get CANDIDATE_TYPE_OPTIONS() {
    return CANDIDATE_OPTIONS;
  }
  get SOURCE_OPTIONS() {
    return SOURCE_OPTIONS;
  }

  selectedFilters: Filters = {};
  constructor(
    private dialogRef: MatDialogRef<FilterViewModalComponent>,
    private subjectService: SharedService,
    @Inject(MAT_DIALOG_DATA)
    public data: {
      type: 'jobs' | 'referrals' | 'applications';
      filters: Filters;
    }
  ) {
    this.selectedFilters = data.filters;
     console.log('My filters', this.selectedFilters);
  }

  closeModal() {
    this.dialogRef.close();
  }

  clearFiler() {
    this.selectedFilters = {
      location: undefined,
      shift: undefined,
      status: undefined,
      experienceLevel: undefined,
      processType: undefined,
    };
    this.subjectService.setFilters({});
    this.dialogRef.close();
  }
  applyFilter() {
    this.dialogRef.close(this.selectedFilters);
    this.subjectService.setFilters(this.selectedFilters);
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
  }
}

filter-view.html

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
    <!-- Job Location -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Job Location</div>
          <button
            *ngIf="selectedFilters.location"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('location')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Location</p>
          <mat-radio-group [(ngModel)]="selectedFilters.location">
            <div *ngFor="let location of LOCATION_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-radio-button
                    class="mt-0 mb-1"
                    [value]="location.value"
                    (change)="onFilterSelect($event, 'location')"
                  ></mat-radio-button>
                </div>
                <div class="right-panel-location">{{ location.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </div>
      </div>
    </div>

    <!-- Timing -->

    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Timing</div>
          <button
            *ngIf="selectedFilters.shift"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('shift')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Timing</p>
          <mat-radio-group [ngModel]="selectedFilters.shift">
            <div *ngFor="let shift of SHIFT_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-radio-button
                    [value]="shift.value"
                    (change)="onFilterSelect($event, 'shift')"
                    class="mt-0 mb-1"
                  ></mat-radio-button>
                </div>
                <div class="right-panel-location">{{ shift.display }}</div>
              </div>
            </div>
          </mat-radio-group>
        </div>
      </div>
    </div>

    <!-- Job Type -->

    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Job Type</div>
          <button
            *ngIf="selectedFilters.processType"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('processType')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Job Type</p>
          <mat-radio-group [ngModel]="selectedFilters.processType">
            <div *ngFor="let processType of PROCESS_TYPE_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-radio-button
                    [value]="processType.value"
                    (change)="onFilterSelect($event, 'processType')"
                    class="mt-0 mb-1"
                  ></mat-radio-button>
                </div>
                <div class="right-panel-location">
                  {{ processType.display }}
                </div>
              </div>
            </div>
          </mat-radio-group>
        </div>
      </div>
    </div>

    <!-- Experience -->
    <div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Experience</div>
          <button
            *ngIf="selectedFilters.experienceLevel"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('experienceLevel')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Experience</p>
          <mat-radio-group [ngModel]="selectedFilters.experienceLevel">
            <div *ngFor="let experienceLevel of EXPERIENCE_LEVEL_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-radio-button
                    [value]="experienceLevel.value"
                    (change)="onFilterSelect($event, 'experienceLevel')"
                    class="mt-0 mb-1"
                  ></mat-radio-button>
                </div>
                <div class="right-panel-location">
                  {{ experienceLevel.display }}
                </div>
              </div>
            </div>
          </mat-radio-group>
        </div>
      </div>
    </div>

    <!-- Source Type -->
    <div class="section row" *ngIf="data.type === 'applications'">
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Source</div>
          <button
            *ngIf="selectedFilters.source"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('source')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Source</p>
          <mat-radio-group [ngModel]="selectedFilters.source">
            <div *ngFor="let source of SOURCE_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-radio-button
                    [value]="source.value"
                    (change)="onFilterSelect($event, 'source')"
                    class="mt-0 mb-1"
                  ></mat-radio-button>
                </div>
                <div class="right-panel-location">
                  {{ source.display }}
                </div>
              </div>
            </div>
          </mat-radio-group>
        </div>
      </div>
    </div>

      <!-- Candidate Type -->
    <div class="section row" *ngIf="data.type === 'applications'">
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Candidate Type</div>
          <button
            *ngIf="selectedFilters.candidateType"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('candidateType')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Candidate Type</p>
          <mat-radio-group [ngModel]="selectedFilters.candidateType">
            <div *ngFor="let candidateType of CANDIDATE_TYPE_OPTIONS">
              <div class="right-panel-content">
                <div class="right-panel-checkbox">
                  <mat-radio-button
                    [value]="candidateType.value"
                    (change)="onFilterSelect($event, 'candidateType')"
                    class="mt-0 mb-1"
                  ></mat-radio-button>
                </div>
                <div class="right-panel-location">
                  {{ candidateType.display }}
                </div>
              </div>
            </div>
          </mat-radio-group>
        </div>
      </div>
    </div>

    <!-- Status -->
    <div
      class="section4 row"
      *ngIf="data.type === 'referrals' || 'applications'"
    >
      <div class="col-5">
        <div class="d-flex align-items-center justify-content-between">
          <div class="filter-job-location">Status</div>
          <button
            *ngIf="selectedFilters.status"
            title="clear filters"
            class="close-btn"
            aria-label="clear filters"
            (click)="onClearFilter('status')"
          >
            <app-icon icon="filter_off"></app-icon>
          </button>
        </div>
      </div>
      <div class="col-7">
        <div class="right-panel">
          <p class="fil-location">Status</p>
          <div *ngFor="let status of STATUS_OPTIONS">
            <div class="right-panel-content">
              <div class="right-panel-checkbox">
                <mat-radio-button
                  [value]="status.value"
                  (change)="onFilterSelect($event, 'status')"
                  class="mt-0 mb-1"
                ></mat-radio-button>
              </div>
              <div class="right-panel-location">{{ status.display }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <mat-card-actions class="footer-button">
    <div class="card-style">
      <div>
        <button
          title="Clear Filter"
          class="ags-outline-btn ags-hxl56 btn-font16 ags-padding1624"
          (click)="clearFiler()"
        >
          Clear Filter
        </button>
      </div>
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
  </mat-card-actions>
</div>

