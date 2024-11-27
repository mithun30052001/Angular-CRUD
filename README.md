https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

req.ts

import { Component, Inject, OnInit } from '@angular/core';
import { MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { ReqService } from '@/src/app/shared/services/req.service';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { Requisition } from '../req-mapping/req-mapping.component';
import { Observable } from 'rxjs';
import { startWith, map } from 'rxjs/operators';

@Component({
  selector: 'app-req-mapping-edit-dialog',
  templateUrl: './req-mapping-edit-dialog.component.html',
  styleUrls: ['./req-mapping-edit-dialog.component.scss'],
})
export class ReqMappingEditDialogComponent implements OnInit {
  reqForm: FormGroup;
  jobOptions: string[] = [];
  filteredJobs: Observable<string[]>;

  hrOptions: string[] = [];
  filteredSPOCs: Observable<string[]>;
  filteredManagers: Observable<string[]>;

  constructor(
    private dialogRef: MatDialogRef<ReqMappingEditDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public rowData: Requisition,
    private reqService: ReqService,
    private fb: FormBuilder
  ) {
    this.reqForm = this.fb.group({
      jobId: ['', Validators.required],
      requisitionId: ['', Validators.required],
      spoc: ['', Validators.required],
      closureDate: ['', Validators.required],
      onboardingDate: ['', Validators.required],
      manager: ['', Validators.required],
    });
  }

  ngOnInit(): void {
    // Fetch job and HR options from services
    this.reqService.getJobData().subscribe((jobs) => {
      this.jobOptions = jobs;
    });

    this.reqService.getHrData().subscribe((hrData) => {
      this.hrOptions = hrData;
    });

    // Set up autocomplete filtering
    this.filteredJobs = this.reqForm.get('jobId')!.valueChanges.pipe(
      startWith(''),
      map((value) => this.filterOptions(value, this.jobOptions))
    );

    this.filteredSPOCs = this.reqForm.get('spoc')!.valueChanges.pipe(
      startWith(''),
      map((value) => this.filterOptions(value, this.hrOptions))
    );

    this.filteredManagers = this.reqForm.get('manager')!.valueChanges.pipe(
      startWith(''),
      map((value) => this.filterOptions(value, this.hrOptions))
    );
  }

  private filterOptions(value: string, options: string[]): string[] {
    const filterValue = value.toLowerCase();
    return options.filter((option) =>
      option.toLowerCase().includes(filterValue)
    );
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.reqForm.valid) {
      const formValues = this.reqForm.value;
      this.reqService.updateJobRequisition(formValues).subscribe(
        (response) => {
          console.log('Job requisition updated successfully:', response);
          this.dialogRef.close(true);
        },
        (error) => {
          console.error('Error updating job requisition:', error);
        }
      );
    }
  }
}




req.html

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
      <!-- Job ID -->
      <div class="form-group ags-form-group">
        <label for="jobId" class="form-label">Job<span class="required"></span></label>
        <mat-form-field appearance="fill">
          <mat-label>Choose a Job</mat-label>
          <input
            matInput
            formControlName="jobId"
            [matAutocomplete]="autoJob"
            placeholder="Enter or select Job ID"
          />
          <mat-autocomplete #autoJob="matAutocomplete">
            <mat-option *ngFor="let job of filteredJobs | async" [value]="job">
              {{ job }}
            </mat-option>
          </mat-autocomplete>
        </mat-form-field>
      </div>

      <!-- Requisition ID -->
      <div class="form-group ags-form-group">
        <label for="requisitionId" class="form-label">Requisition ID<span class="required"></span></label>
        <mat-form-field appearance="fill">
          <mat-label>Requisition ID</mat-label>
          <input matInput formControlName="requisitionId" placeholder="Enter requisition ID" />
        </mat-form-field>
      </div>

      <!-- SPOC -->
      <div class="form-group ags-form-group">
        <label for="spoc" class="form-label">SPOC<span class="required"></span></label>
        <mat-form-field appearance="fill">
          <mat-label>Choose SPOC</mat-label>
          <input
            matInput
            formControlName="spoc"
            [matAutocomplete]="autoSPOC"
            placeholder="Enter or select SPOC"
          />
          <mat-autocomplete #autoSPOC="matAutocomplete">
            <mat-option *ngFor="let spoc of filteredSPOCs | async" [value]="spoc">
              {{ spoc }}
            </mat-option>
          </mat-autocomplete>
        </mat-form-field>
      </div>

      <!-- Closure Date -->
      <div class="form-group ags-form-group">
        <label for="closureDate" class="form-label">Closure Date<span class="required"></span></label>
        <mat-form-field appearance="fill">
          <mat-label>Closure Date</mat-label>
          <input matInput formControlName="closureDate" type="date" />
        </mat-form-field>
      </div>

      <!-- Onboarding Date -->
      <div class="form-group ags-form-group">
        <label for="onboardingDate" class="form-label">Onboarding Date<span class="required"></span></label>
        <mat-form-field appearance="fill">
          <mat-label>Onboarding Date</mat-label>
          <input matInput formControlName="onboardingDate" type="date" />
        </mat-form-field>
      </div>

      <!-- Manager -->
      <div class="form-group ags-form-group">
        <label for="manager" class="form-label">Manager<span class="required"></span></label>
        <mat-form-field appearance="fill">
          <mat-label>Choose Manager</mat-label>
          <input
            matInput
            formControlName="manager"
            [matAutocomplete]="autoManager"
            placeholder="Enter or select Manager"
          />
          <mat-autocomplete #autoManager="matAutocomplete">
            <mat-option *ngFor="let manager of filteredManagers | async" [value]="manager">
              {{ manager }}
            </mat-option>
          </mat-autocomplete>
        </mat-form-field>
      </div>
    </div>

    <div class="req-edit-footer">
      <button mat-dialog-close class="ags-outline-btn">Cancel</button>
      <button type="submit" [disabled]="reqForm.invalid" class="ags-primary-btn">Submit</button>
    </div>
  </form>
</div>
