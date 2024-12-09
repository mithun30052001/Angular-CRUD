https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

interview-summary-dialog.ts

import { CandidateApplication, Quiz } from '@/src/app/interfaces/app-interface';
import { Component, Inject } from '@angular/core';
import { FormControl, FormGroup, Validators } from '@angular/forms';
import {
  MatDialog,
  MatDialogRef,
  MAT_DIALOG_DATA,
} from '@angular/material/dialog';
import { capitalize } from 'lodash-es';
import { PANEL_STATUS_OPTIONS } from '@/src/app/constants/app.constants';
import { ApplicationService } from '../../services/job-application.service';
import { MessageModalsComponent } from '@/src/app/common-modules/message-modals/message-modals.component';

@Component({
  selector: 'app-interview-summary-dialog',
  templateUrl: './interview-summary-dialog.component.html',
  styleUrls: ['./interview-summary-dialog.component.scss'],
})
export class InterviewSummaryDialogComponent {
  form = new FormGroup({
    status: new FormControl(''),
    feedBack: new FormControl(''),
  });
  formBuilder: any;
  mobileView = false;

  constructor(
    private dialogRef: MatDialogRef<InterviewSummaryDialogComponent>,
    public dialog: MatDialog,
    private applicationService: ApplicationService,
    @Inject(MAT_DIALOG_DATA)
    public summary: {
      application: CandidateApplication;
      interviewProcess: any[];
    }
  ) {
    console.log(this.summary);
  }

  get PANEL_STATUS_OPTIONS() {
    return PANEL_STATUS_OPTIONS;
  }

  ngOnInit(): void {
    this.initiateForm();
  }

  initiateForm() {
    const interview = this.summary.interviewProcess;
    console.log("MY SUMMARY INSIDE INIT", this.summary);
    this.form = this.formBuilder.group({
      feedBack: [
        {
          value: '',
          disabled: false,
        },
        Validators.required,
      ],
      status: [
        {
          value: '',
          disabled: false,
        },
        Validators.required,
      ],
    });
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.form.valid) {
      console.log('FORM VALUES', this.form.value);
      this.applicationService
        .updateApplicationStatus(this.summary?.application?.id, this.form.value)
        .then((_res) => {
          // setTimeout(() => {
          this.form.reset();
          this.dialog.open(MessageModalsComponent, {
            maxWidth: this.mobileView ? '500px' : '100%',
            width: this.mobileView ? '100%' : '50%',
            ...(this.mobileView && {
              height: '450px',
              position: { bottom: '0' },
            }),
            closeOnNavigation: true,
            disableClose: true,
            data: 'panelUpdate',
          });
          // }, 2000);
        });
    } else {
      console.log('FORM IS VALID');
    }
  }

  getStatus(status: 'COMPLETED' | 'PROGRESS' | 'FAILED') {
    if (status === 'COMPLETED') return 'Selected';
    if (status === 'PROGRESS') return 'In Progress';
    if (status === 'FAILED') return 'Rejected';
    else return capitalize(status);
  }

  calculatePercentage(quiz: Quiz) {
    let percentage = 0;
    const partial = quiz.score;
    const total = quiz.noOfQuestions;
    if (!partial || !total) return '0%';
    percentage = (100 * partial) / total;

    return `${percentage}%`;
  }
}



interview-details.component.html

 <ng-container
            *ngIf="
              interview.candidates[0].interviewStatus === 'PROGRESS';
              else elsePart
            "
          >
            <form [formGroup]="form">
              <div class="d-flex justify-content-start">
                <div class="mt-3 form-group form-inner">
                  <p class="assment-subheading">
                    Status:&nbsp;&nbsp;
                    <mat-form-field class="example-form-field">
                      <mat-select
                        placeholder="Select status"
                        formControlName="status"
                      >
                        <mat-option
                          [value]="option.value"
                          *ngFor="let option of PANEL_STATUS_OPTIONS"
                          >{{ option.display }}</mat-option
                        >
                      </mat-select>
                    </mat-form-field>
                  </p>
                </div>
              </div>
              <div class="form-inner mt-3">
                <label class="panel-feedback" for="feedBack"
                  >Panel remarks</label
                >
                <textarea
                  formControlName="feedBack"
                  placeholder="Panel Feedback"
                  class="text-area"
                ></textarea>
              </div>
            </form>
          </ng-container>

          <ng-template #elsePart>
            <p class="assment-subheading">
              Status:
              <span
                [ngClass]="{
                  'active-green': interview.candidates[0].interviewStatus === 'COMPLETED',
                  'active-warn': interview.candidates[0].interviewStatus === 'PROGRESS',
                  'active-error': ['REJECTED', 'FAILED'].includes(
                    interview.candidates[0].interviewStatus
                  )
                }"
                >{{ getStatus(interview.candidates[0].interviewStatus) }}</span
              >
            </p>
            <div *ngIf="interview.candidates[0].feedback" class="form-inner mt-3">
              <textarea
                readonly
                [value]="interview.candidates[0].feedback"
                class="text-area"
              ></textarea>
            </div>
          </ng-template>
