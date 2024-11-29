https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req-filter.component.ts

import { Component, Inject, isDevMode } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialogRef } from '@angular/material/dialog';
import { SharedService } from '@/src/app/subject-module/shared.service';
import { Filters } from '@/src/app/interfaces/app-interface';
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
    private sharedService: SharedService,
    private router: Router,
    @Inject(MAT_DIALOG_DATA)
    public data: { type: 'jobs' | 'referrals' | 'applications'; filters: any }
  ) {
    this.selectedFilters = data.filters || {};
  }

  closeModal() {
    this.dialogRef.close();
  }

  clearFilter() {
    this.selectedFilters = {};
    this.sharedService.setFilters({});
    this.dialogRef.close();
  }

  applyFilter() {
    // Encode filters as Base64 and add to query params
    const filterString = JSON.stringify(this.selectedFilters);
    const base64Filters = btoa(filterString);

    this.router.navigate([], {
      queryParams: { filters: base64Filters },
      queryParamsHandling: 'merge',
    });

    this.dialogRef.close(this.selectedFilters);
    this.sharedService.setFilters(this.selectedFilters);
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

<!-- Example for Request Type -->
<div class="section row" *ngIf="data.type === 'jobs' || 'applications'">
  <div class="col-sm-3">
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
  <div class="col-sm-9">
    <div class="right-panel">
      <div class="row">
        <div class="col-sm-3" *ngFor="let request_type of REQUEST_TYPE_OPTIONS">
          <div class="right-panel-content">
            <div class="right-panel-checkbox">
              <mat-checkbox
                class="mt-0 mb-1"
                [checked]="selectedFilters.request_type?.includes(request_type.value)"
                (change)="onFilterSelect(request_type.value, 'request_type')"
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


component.ts

import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-another-component',
  templateUrl: './another-component.component.html',
  styleUrls: ['./another-component.component.scss'],
})
export class AnotherComponent implements OnInit {
  filters: any;

  constructor(private route: ActivatedRoute) {}

  ngOnInit() {
    this.route.queryParams.subscribe((params) => {
      if (params['filters']) {
        const decodedFilters = atob(params['filters']);
        this.filters = JSON.parse(decodedFilters);
        console.log('Decoded filters:', this.filters);
      }
    });
  }
}
