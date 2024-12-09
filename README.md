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
  form: FormGroup;
  formBuilder: any;
  mobileView = false;

  constructor(
    private dialogRef: MatDialogRef<InterviewSummaryDialogComponent>,
    public dialog: MatDialog,
    private applicationService: ApplicationService,
    @Inject(MAT_DIALOG_DATA)
    public summary: {
      application: CandidateApplication;
      applicationStatus: any,
      interviewProcess: any[];
    }
  ) {
    console.log(this.summary.applicationStatus);
  }

  get PANEL_STATUS_OPTIONS() {
    return PANEL_STATUS_OPTIONS;
  }

  ngOnInit(): void {
    this.initiateForm();
  }

  // initiateForm() {
  //   const interview = this.summary.interviewProcess;
  //   console.log("MY SUMMARY INSIDE INIT", this.summary);
  //   this.form = this.formBuilder.group({
  //     feedBack: [
  //       {
  //         value: '',
  //         disabled: false,
  //       },
  //       Validators.required,
  //     ],
  //     status: [
  //       {
  //         value: '',
  //         disabled: false,
  //       },
  //       Validators.required,
  //     ],
  //   });
  // }

  initiateForm() {
    const interview = this.summary.interviewProcess[0];
    const firstCandidate = interview.candidates[0];

    this.form = new FormGroup({
      feedBack: new FormControl('', Validators.required),
      status: new FormControl(this.summary?.applicationStatus ?? '', Validators.required),
    });

    console.log('FORM INITIALIZED WITH', firstCandidate);
    console.log('FORM INITIALIZED VALUES', this.form.value);
  }

  closeModal() {
    this.dialogRef.close();
  }

  onSubmit() {
    if (this.form.valid) {
      console.log('FORM VALUES', this.form.value);
      // this.applicationService
      //   .updateApplicationStatus(this.summary?.application?.id, this.form.value)
      //   .then((_res) => {
      //     // setTimeout(() => {
      //     this.form.reset();
      //     this.dialog.open(MessageModalsComponent, {
      //       maxWidth: this.mobileView ? '500px' : '100%',
      //       width: this.mobileView ? '100%' : '50%',
      //       ...(this.mobileView && {
      //         height: '450px',
      //         position: { bottom: '0' },
      //       }),
      //       closeOnNavigation: true,
      //       disableClose: true,
      //       data: 'panelUpdate',
      //     });
      //     // }, 2000);
      //   });
    } else {
      console.log('FORM IS INVALID');
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
