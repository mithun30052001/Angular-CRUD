https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

refer-buddy.component.ts

import { MediaMatcher } from '@angular/cdk/layout';
import { Component, Inject, OnInit, ViewChild } from '@angular/core';
import {
  AbstractControl,
  FormBuilder,
  FormGroup,
  Validators,
} from '@angular/forms';
import {
  MAT_DIALOG_DATA,
  MatDialog,
  MatDialogRef,
} from '@angular/material/dialog';
import { ProfileService } from '../../shared/services/profile.service';
import { UploadService } from '../file-upload/upload.service';
import { MessageModalsComponent } from '../message-modals/message-modals.component';
import { ResponsiveViewComponent } from '../../shared/components/responsive-view/responsive-view.component';
import { NgxMatIntlTelInputComponent } from 'ngx-mat-phone-input';

// const phoneRegex = /^((\\+91-?)|0)?[0-9]{10}$/g;

@Component({
  selector: 'app-refer-buddy',
  templateUrl: './refer-buddy.component.html',
  styleUrls: ['./refer-buddy.component.scss'],
})
export class ReferBuddyComponent
  extends ResponsiveViewComponent
  implements OnInit
{
  referBuddy: FormGroup | any;

  isChecked = true;
  formValue: any = {};
  fileName = '';
  fileSize = '';
  lastUpdatedFile = '';
  uploadPercentage = '';
  file: File | any;
  fileResponse: any;
  resumeDeleted = false;
  userID = '';
  jobId: string;
  @ViewChild('mob')
  mob!: NgxMatIntlTelInputComponent;

  get f(): { [key: string]: AbstractControl } {
    return this.referBuddy.controls;
  }

  constructor(
    private formBuilder: FormBuilder,
    media: MediaMatcher,
    private profileService: ProfileService,
    public dialog: MatDialog,
    private dialogRef: MatDialogRef<ReferBuddyComponent>,
    private uploadService: UploadService,
    @Inject(MAT_DIALOG_DATA) public data: { jobId: string }
  ) {
    super(media);
    console.log(data);
    this.jobId = data?.jobId;
    this.userID = localStorage.getItem('id') || '';
  }
  ngOnInit() {
    this.referBuddy = this.formBuilder.group({
      firstName: ['', Validators.required],
      lastName: [''],
      email: [
        '',
        [
          Validators.required,
          Validators.pattern(
            /^([a-zA-Z0-9_.-]){1,50}@(([a-zA-Z0-9-]){1,50}\.)+([a-zA-Z]{2,50})+$/
          ),
        ],
      ],
      mobile: ['', [Validators.required]],
    });
  }

  async submitForm() {
    this.formValue = this.referBuddy.value;
    const candidateProfile = {
      fullName: this.formValue.firstName + ' ' + this.formValue.lastName,
      firstName: this.formValue.firstName,
      lastName: this.formValue.lastName,
      email: this.formValue.email,
      mobile: this.f['mobile'].value.replace(
        `+${this.mob.selectedCountry?.dialCode}`,
        ''
      ),
      countryCode: `+${this.mob.selectedCountry?.dialCode}`,
      preferredWalkinLocation: 'Chennai',
      experienceLevel: 'Fresher',
      referrer: this.userID,
      ...(this.jobId && { jobId: this.jobId }),
    };
    const createCandidateResponse =
      await this.profileService.createCandidateProfile(candidateProfile);
    if (createCandidateResponse.id) {
      this.dialog
        .open(MessageModalsComponent, {
          maxWidth: this.mobileView ? '500px' : '100%',
          width: this.mobileView ? '100%' : '30%',
          ...(this.mobileView && { height: 'auto', position: { bottom: '0' } }),
          closeOnNavigation: true,
          disableClose: true,
          data: this.mobileView
            ? 'referBuddySuccessfullyMobile'
            : 'referBuddySuccessfullyWeb',
        })
        .afterClosed()
        .subscribe((_result: any) => {
          if (window.location.pathname === '/c/referral-status') {
            window.location.reload();
          } else {
            if (window.location.pathname === '/e/employee-referrals') {
              window.location.reload();
            }
          }
        });
    } else {
      this.dialog.open(MessageModalsComponent, {
        maxWidth: this.mobileView ? '500px' : '100%',
        width: this.mobileView ? '100%' : '30%',
        ...(this.mobileView && { height: 'auto', position: { bottom: '0' } }),
        closeOnNavigation: true,
        disableClose: true,
        data: this.mobileView ? 'errorOccurMobile' : 'errorOccurWeb',
      });
    }
    this.dialogRef.close();
  }

  disableForm() {
    return this.referBuddy.status !== 'VALID';
  }

  async droppedFiles(allFiles: any) {
    this.fileName = '';
    this.resumeDeleted = false;
    this.file = allFiles.target.files[0];
    const uploadResponse = await this.uploadService.uploadFile(
      this.file,
      this.userID
    );
    if (uploadResponse) {
      this.fileResponse = uploadResponse;
      this.fileName = this.file.name;
      this.fileSize = '1MB';
      this.lastUpdatedFile = '03/08/2023';
      this.uploadPercentage = '100 % uploaded';
    }
  }

  async deleteResume() {
    const fileDeleResponse = await this.uploadService.deleteFile(this.userID);
    if (fileDeleResponse.message === 'SUCCESS') {
      this.resumeDeleted = true;
      this.fileName = 'None';
    }
  }
  closeModal() {
    this.dialogRef.close();
  }
}


refer-buddy.component.html

 <div class="input-boxes form-inner col-sm-6">
          <label class="required" for="mob">Mobile</label>
          <ngx-mat-intl-tel-input
            #mob
            format="default"
            [enablePlaceholder]="true"
            inputPlaceholder="Enter mobile number"
            formControlName="mobile"
            [onlyCountries]="['us', 'ph', 'in', 'mx']"
            class="number-field position-relative"
          >
          </ngx-mat-intl-tel-input>
          <div
            *ngIf="f['mobile'].dirty && f['mobile'].touched && f['mobile']?.errors?.['required']"
            class="form-invalid"
          >
            <p>Mobile number is required</p>
          </div>
          <div
            *ngIf="f['mobile'].dirty && f['mobile']?.errors?.['validatePhoneNumber']"
            class="form-invalid"
          >
            <p>Mobile number is invalid</p>
          </div>
        </div>
