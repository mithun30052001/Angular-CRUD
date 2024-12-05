https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

interview-summary-dialog.component.ts

import { CandidateApplication, Quiz } from '@/src/app/interfaces/app-interface';
import { Component, Inject } from '@angular/core';
import { FormControl, FormGroup, Validators } from '@angular/forms';
import {
  MatDialog,
  MatDialogRef,
  MAT_DIALOG_DATA,
} from '@angular/material/dialog';
import { capitalize } from 'lodash-es';

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

  constructor(
    private dialogRef: MatDialogRef<InterviewSummaryDialogComponent>,
    public dialog: MatDialog,
    @Inject(MAT_DIALOG_DATA)
    public summary: {
      application: CandidateApplication;
      interviewProcess: any[];
    }
  ) {
    console.log(this.summary);
  }

  ngOnInit(): void {
    this.initiateForm();
  }

  initiateForm() {
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


interview-summary-dialog.component.html

<div class="interview-summary-modal">
  <div class="interview-summary-header">
    <h5 class="heading-text">Interview summary</h5>
    <div>
      <app-icon icon="close" (click)="closeModal()"></app-icon>
    </div>
  </div>
  <div class="interview-summary-body">
    <ng-container
      *ngFor="let interview of summary.interviewProcess; let i = index"
    >
      <div *ngIf="interview.interviewType === 'HR_ROUND'" class="hr-interview">
        <div class="d-flex column-gap-3 mb-3">
          <span class="sno">{{ i + 1 }}</span>
          <div class="interview-title">HR Feedback</div>
        </div>
        <div class="form-group ags-form-group">
          <textarea
            readonly
            [value]="interview.feedBack"
            class="comment form-control"
          >
          </textarea>
        </div>
        <mat-divider></mat-divider>
      </div>

      <div
        *ngIf="['ASSESMENT', 'VERSANT'].includes(interview.interviewType)"
        class="assessment-interview"
      >
        <div class="d-flex justify-content-between">
          <div class="d-flex column-gap-3">
            <span class="sno">{{ i + 1 }}</span>
            <div class="interview-title">
              {{
                interview.interviewType === 'ASSESMENT'
                  ? 'MAC Interview'
                  : 'Versant Interview'
              }}
            </div>
          </div>
          <div
            *ngIf="interview.interviewStatus === 'COMPLETED'"
            class="score-container"
          >
            <div *ngIf="interview?.grade" class="grade-container">
              <p class="grade-cont">
                Grade :
                <span class="grade">{{ interview.grade | textFill }}</span>
              </p>
            </div>
            <div *ngIf="interview?.score" class="score-container">
              <p class="score-label">Score</p>
              <div class="position-relative">
                <mat-spinner
                  [diameter]="40"
                  mode="determinate"
                  [value]="interview.score"
                ></mat-spinner>
                <span class="circle-value">{{
                  interview.score | textFill
                }}</span>
              </div>
            </div>
          </div>
        </div>
        <div class="d-flex justify-content-between column-gap-5">
          <div class="mt-3">
            <p class="assment-subheading">
              LOB :
              <span>
                {{
                  summary.application.job.processType | textFill | sentenceCase
                }}
                -
                {{ summary.application.experience | textFill | sentenceCase }}
              </span>
            </p>
            <ng-container *ngFor="let quiz of interview.quizDetails">
              <div class="d-flex column-gap-2">
                <p class="assment-subheading">{{ quiz.quizName }} Score:</p>
                <p
                  [ngClass]="{
                  'active-green':
                    quiz.status === 'COMPLETED',
                  'active-warn': quiz.status === 'PROGRESS',
                  'active-error': quiz.status === 'FAILED',
                }"
                >
                  {{ calculatePercentage(quiz) }}
                </p>
              </div>
              <div
                *ngIf="quiz.quizName === 'Typing test'"
                class="d-flex column-gap-2"
              >
                <p class="assment-subheading">Words per minute:</p>
                <p
                  [ngClass]="{
                  'active-green':
                    quiz.status === 'COMPLETED',
                  'active-warn': quiz.status === 'PROGRESS',
                  'active-error': quiz.status === 'FAILED',
                }"
                >
                  {{ quiz.wpm }}
                </p>
              </div>
            </ng-container>
          </div>

          <div class="mt-3">
            <p class="assment-subheading">
              Status :
              <span
                [ngClass]="{
                  'active-green': interview.interviewStatus === 'COMPLETED',
                  'active-warn': interview.interviewStatus === 'PROGRESS',
                  'active-error': ['REJECTED', 'FAILED'].includes(
                    interview.interviewStatus
                  )
                }"
                >{{ getStatus(interview.interviewStatus) }}</span
              >
            </p>
            <div *ngIf="interview?.tin">
              <span class="system-allocate">Tin: </span>
              <p class="system-number">
                {{ interview.tin }}
              </p>
            </div>
          </div>
        </div>
        <div *ngIf="interview.FeedBack" class="form-inner">
          <textarea
            readonly
            [value]="interview.feedBack ? interview.feedBack : ''"
            class="text-area"
          ></textarea>
        </div>
        <mat-divider></mat-divider>
      </div>

      <div *ngIf="interview.interviewType === 'PANEL'" class="panel-interview">
        <div class="d-flex justify-content-between">
          <div class="d-flex column-gap-3">
            <span class="sno">{{ i + 1 }}</span>
            <div class="interview-title">Panel Interview</div>
          </div>
        </div>
        <div class="d-flex justify-content-between">
          <div class="mt-3">
            <p class="panel-heading">Employees for the panel</p>

            <ng-container *ngIf="interview.interviewStatus === 'PROGRESS'">
              <p
                *ngFor="let member of interview.panelMembers"
                [title]="member.email | textFill"
                class="assment-subheading"
              >
                {{ member.name | textFill }}
                <mat-chip *ngIf="member.primaryPanelMember">primary</mat-chip>
              </p>
            </ng-container>
            <ng-container
              *ngIf="
                interview?.panelUpdatedEmployee?.name &&
                interview.interviewStatus !== 'PROGRESS'
              "
            >
              <p>
                {{ interview?.panelUpdatedEmployee?.name }}
              </p>
            </ng-container>
          </div>
          <div class="mt-3">
            <p class="panel-heading">Scheduled date</p>
            <p class="panel-date">
              {{ interview.startTime | date : 'dd MMM YYYY,' }}
              {{ interview.startTime | date : 'hh:mma' }} -
              {{ interview.endTime | date : 'hh:mma' }}
            </p>
          </div>
        </div>

        <div class="d-flex justify-content-start">
          <div class="mt-3">
            <p class="assment-subheading">
              Status :
              <span
                [ngClass]="{
                  'active-green': interview.interviewStatus === 'COMPLETED',
                  'active-warn': interview.interviewStatus === 'PROGRESS',
                  'active-error': ['REJECTED', 'FAILED'].includes(
                    interview.interviewStatus
                  )
                }"
                >{{ getStatus(interview.interviewStatus) }}</span
              >
            </p>
          </div>
        </div>
        <div class="form-inner">
          <textarea
            readonly
            [value]="
              interview?.candidates[0]?.feedback
                ? interview.candidates[0] .feedback
                : ''
            "
            class="text-area"
          ></textarea>
        </div>

        <mat-divider></mat-divider>
      </div>

      <div
        *ngIf="interview.interviewType === 'HR_REJECTED'"
        class="reject-reason"
      >
        <div class="d-flex justify-content-between">
          <div class="d-flex column-gap-3 mb-3">
            <span class="sno">{{ i + 1 }}</span>
            <div class="interview-title">Reason for reject</div>
          </div>
          <div class="">
            <p class="panel-heading">
              Rejected by :
              <span class="assment-subheading">{{
                interview.hr.name | textFill
              }}</span>
            </p>
          </div>
        </div>

        <div class="form-group ags-form-group">
          <input
            readonly
            [value]="interview.feedBack | sentenceCase"
            class="comment form-control"
          />
        </div>
      </div>
    </ng-container>
  </div>
</div>


form data
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

