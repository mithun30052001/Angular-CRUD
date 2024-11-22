https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

profile.ts

uploadProfileImage(file: File) {
    this.profileService.uploadProfileImage(this.userId, file).subscribe({
      next: (response) => {
        console.log('Profile image uploaded successfully', response);
        this.getProfileImage();  // Refresh the image preview after upload
      },
      error: (error) => {
        console.error('Error uploading image:', error);
      },
    });
  }

  getProfileImage() {
    this.profileService.getProfileImage(this.userId).subscribe({
      next: (response) => {
        // Create an image URL from the blob response
        const reader = new FileReader();
        reader.onloadend = () => {
          this.imageUrl = reader.result as string;
        };
        reader.readAsDataURL(response);
      },
      error: (error) => {
        console.error('Error fetching profile image:', error);
      },
    });
  }

  openImagePreview() {
    const dialogRef = this.dialog.open(ImagePreviewDialog, {
      data: { imageUrl: this.imageUrl },
    });

    dialogRef.afterClosed().subscribe((result) => {
      if (result === 'delete') {
        this.deleteProfileImage();
      }
    });
  }

  deleteProfileImage() {
    this.profileService.deleteProfileImage(this.userId).subscribe({
      next: (response) => {
        console.log('Profile image deleted successfully', response);
        this.imageUrl = 'assets/images/profile-pic.png';  // Reset to default image
      },
      error: (error) => {
        console.error('Error deleting image:', error);
      },
    });
  }

profile-service
private baseUrl = `${environment.API_URL}/candidate-services/profile`;

  constructor(private httpClient: HttpClient) {}

  // Upload Profile Image
  uploadProfileImage(id: string, file: File): Observable<any> {
    const formData = new FormData();
    formData.append('image', file, file.name);

    return this.httpClient.post(`${this.baseUrl}/${id}/image`, formData);
  }

  // Get Profile Image
  getProfileImage(id: string): Observable<any> {
    return this.httpClient.get(`${this.baseUrl}/${id}/image`, { responseType: 'blob' });
  }

  // Delete Profile Image
  deleteProfileImage(id: string): Observable<any> {
    return this.httpClient.delete(`${this.baseUrl}/${id}/image`);
  }

quickEdit()

quickEdit() {
    this.dialog
      .open(CandidateQuickEditComponent, {
        maxWidth: this.mobileView ? '500px' : '100%',
        width: this.mobileView ? '100%' : '50%',
        ...(this.mobileView && { height: 'auto', position: { bottom: '0%' } }),
        closeOnNavigation: true,
        disableClose: true,
        ...(this.mobileView ? {} : { panelClass: 'dialog-responsive' }),
      })
      .afterClosed()
      .subscribe(() => {
        this.getProfile();
      });
  }

  candidate-quick-edit.html

<!-- Web View -->

<div class="quick-edit-modal">
  <div class="quick-edit-header">
    <form [formGroup]="editProfile" novalidate>
      <div class="heading">
        <div class="heading-text">Edit Quick Info</div>
        <button class="icon-button" (click)="closeModal()">
          <app-icon icon="close"></app-icon>
        </button>
      </div>

      <div class="row">
        <div class="col-sm-12">
          <div class="form-group ags-form-group">
            <label class="form-label" for="email">Email</label>
            <input
              matInput
              formControlName="emailID"
              placeholder="Enter your Email id"
              [errorStateMatcher]="matcher"
              class="form-control"
            />
            <div class="ags-invalid" *ngIf="f['emailID']?.errors?.['required']">
              Required
            </div>
            <div class="ags-invalid" *ngIf="f['emailID'].invalid">
              Email is invalid
            </div>
          </div>
        </div>
        <div class="col-sm-12">
          <div class="input-boxes form-inner">
            <div class="d-flex align-items-center justify-content-between">
              <label for="mob">Mobile</label>
            </div>
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
        </div>
        <div class="col-sm-12">
          <div class="form-group form-inner">
            <label for="dob" class="form-label"
              >Date Of Birth<span class="required"></span
            ></label>

            <div class="form-datepicker">
              <input
                (dateChange)="onDateChange($event)"
                #dob
                readonly
                [max]="max"
                formControlName="dob"
                class="form-control"
                [matDatepicker]="picker"
              />
              <mat-datepicker-toggle
                matIconSuffix
                [for]="picker"
              ></mat-datepicker-toggle>
            </div>
            <mat-datepicker #picker></mat-datepicker>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-sm-12">
          <div class="form-group ags-form-group">
            <!-- <label class="required form-label" for="otp">OTP</label> -->
            <div class="d-flex align-items-center justify-content-between">
              <label for="mob">OTP</label>
              <div aria-hidden="true" class="send-otp" (click)="sendOTP()">
                Send OTP
              </div>
            </div>
            <div class="input-container">
              <input
                type="password"
                [type]="hide ? 'password' : 'text'"
                matInput
                formControlName="otp"
                class="form-control"
                placeholder="Enter your OTP"
              />
              <app-icon
                class="app-icon"
                [icon]="hide ? 'visibility_off' : 'visibility'"
                (click)="hide = !hide"
              ></app-icon>
            </div>
            <mat-hint *ngIf="otpSend" class="w-100"
              ><div class="row">
                <div class="col-8 mobile-hint-text mt-1 tx-color">
                  OTP sent successfully
                </div>
              </div>
            </mat-hint>
            <div
              class="ags-invalid"
              *ngIf="f['otp'].dirty && f['otp'].touched && f['otp'].invalid"
            >
              OTP is required
            </div>
          </div>
        </div>
      </div>
    </form>
  </div>

  <div class="d-flex justify-content-md-end quick-edit-footer">
    <div class="mr-1rem">
      <button
        title="Cancel"
        mat-dialog-close
        class="ags-outline-btn ags-hxl56 btn-font16 ags-padding1624"
      >
        Cancel
      </button>
    </div>
    <div>
      <button
        title="Save"
        class="ags-primary-btn ags-hxl56 btn-font16 ags-padding1624"
        (click)="submitForm()"
        [disabled]="disableForm()"
      >
        Save
      </button>
    </div>
  </div>
</div>




profile_picture

<div class="row d-flex align-items-center">
  <div class="col-sm-4 col-lg-2 col-3">
    <div class="profile-avator position-relative">
      <img src="assets/images/profile-pic.png" alt="" class="img-fluid" />
      
      <button class="icon-button position-absolute bottom-0 start-0 mb-2 ms-2" (click)="openFileSelector()">
        <app-icon icon="edit_ic" title="Edit Profile Picture" />
      </button>
      
      <input type="file" #fileInput class="d-none" (change)="onFileSelected($event)" accept="image/*" />
    </div>
  </div>
  <div class="col-sm-7 col-lg-9 col-8">
    <div class="profile-left-heading">
      <p class="">{{ profile.fullName }}</p>
      <p class="">{{ profile.mobile }}</p>
      <p class="mail-color">{{ profile.email }}</p>
    </div>
  </div>
  <div class="col-sm-1 col-lg-1 col-1 d-flex justify-content-end">
    <div class="profile-heading-edit">
      <button class="icon-button" (click)="quickEdit()">
        <app-icon icon="edit_ic" title="Quick edit" />
      </button>
    </div>
  </div>
</div>


profile.component.ts

import {
  Component,
  ElementRef,
  EventEmitter,
  Input,
  OnInit,
  Output,
  ViewChild,
} from '@angular/core';
import { MatTabChangeEvent, MatTabGroup } from '@angular/material/tabs';
import { ActivatedRoute, Router } from '@angular/router';
import { ProfileDetailsComponent } from './profile-details/profile-details.component';
import { WorkExperienceComponent } from './work-experience/work-experience.component';
import { PersonalInformationComponent } from './personal-information/personal-information.component';
import { EducationDetailsComponent } from './education-details/education-details.component';
import { WorkDetailsComponent } from './work-details/work-details.component';
import { IdProofsComponent } from './id-proofs/id-proofs.component';
// import { FitmentsComponent } from './fitments/fitments.component';
import { ProfileService } from '@/src/app/shared/services/profile.service';
import { ResponsiveViewComponent } from '@/src/app/shared/components/responsive-view/responsive-view.component';
import { MediaMatcher } from '@angular/cdk/layout';
import {
  Address,
  EducationDetails,
  // FitmentDetails,
  IdProof,
  PersonalInformation,
  // ProcessType,
  ProfileDetails,
  WorkDetails,
  WorkExperience,
} from '@/src/app/interfaces/app-interface';
import { FormControl } from '@angular/forms';
import { SharedOutputService } from '@/src/app/shared/services/shared-output.service';
import { CandidateQuickEditComponent } from '../../candidate/components/candidate-profile-menu/candidate-quick-edit/candidate-quick-edit.component';
import { MatDialog } from '@angular/material/dialog';

interface Content {
  profile: { id: string } & ProfileDetails;
  work: { id: string } & WorkExperience;
  workExp: { id: string; workDetails: WorkDetails[] };
  personalInfo: {
    id: string;
    addressDetails: Address[];
    maritalStatus: string;
  } & PersonalInformation;
  idProof: {
    id: string;
    idProof: IdProof[];
    firstName: string;
    lastName: string;
  };
  // fitment: { id: string; lob: ProcessType } & FitmentDetails;
  education: { id: string; educationDetails: EducationDetails[] };
}
@Component({
  selector: 'app-profile',
  templateUrl: './profile.component.html',
  styleUrls: ['./profile.component.scss'],
})
export class ProfileComponent
  extends ResponsiveViewComponent
  implements OnInit
{
  @ViewChild('tab', { static: false }) tab: MatTabGroup;
  @ViewChild('fileInput') fileInput: ElementRef;

  queryParams = {
    tab: 0,
  };

  content: Content;

  @Input() userId?: string;
  @Output() save = new EventEmitter<boolean>(false);

  originalList: { id: keyof Content; label: string; content: any }[] = [
    {
      id: 'profile',
      label: 'Profile Details',
      content: ProfileDetailsComponent,
    },
    { id: 'work', label: 'Work Details', content: WorkDetailsComponent },
    {
      id: 'education',
      label: 'Education Details',
      content: EducationDetailsComponent,
    },
    {
      id: 'personalInfo',
      label: 'Address Information',
      content: PersonalInformationComponent,
    },
    {
      id: 'workExp',
      label: 'Employment History',
      content: WorkExperienceComponent,
    },
    // { id: 'fitment', label: 'Fitments', content: FitmentsComponent },

    { id: 'idProof', label: 'ID proof', content: IdProofsComponent },
  ];

  tabContents: { id: keyof Content; label: string; content: any }[] = [
    {
      id: 'profile',
      label: 'Profile Details',
      content: ProfileDetailsComponent,
    },
    { id: 'work', label: 'Work Details', content: WorkDetailsComponent },
    {
      id: 'education',
      label: 'Education Details',
      content: EducationDetailsComponent,
    },
    {
      id: 'personalInfo',
      label: 'Address Information',
      content: PersonalInformationComponent,
    },
    {
      id: 'workExp',
      label: 'Employment History',
      content: WorkExperienceComponent,
    },
    // { id: 'fitment', label: 'Fitments', content: FitmentsComponent },

    { id: 'idProof', label: 'ID proof', content: IdProofsComponent },
  ];

  selectedTab = new FormControl(0);

  profile: { id: string } & ProfileDetails;
  workExperienceDetails: { id: string } & WorkExperience;
  workDetails: ({ id: string } & WorkDetails)[];
  personalInfoDetails: { id: string } & PersonalInformation;
  idProofDetails: { id: string; idProof: IdProof[] };
  // fitmentDetails: { id: string } & FitmentDetails;
  educationDetails: { id: string } & EducationDetails;
  isHr = '';

  keys: string[] = [
    'profile',
    'education',
    'personalInfo',
    'workExp',
    'work',
    // 'fitment',
    'idProof',
  ];

  constructor(
    media: MediaMatcher,
    private router: Router,
    private activatedRoute: ActivatedRoute,
    private profileService: ProfileService,
    private so: SharedOutputService,
    private dialog: MatDialog
  ) {
    super(media);

    this.activatedRoute.queryParams.subscribe((params) => {
      this.queryParams = params as { tab: number };
    });

    if (!this.userId) {
      this.userId = localStorage.getItem('id') as string;
    }
    this.isHr = localStorage.getItem('role') as string;
  }

  getTabInput(id: keyof Content) {
    return {
      initialValues: this.content?.[id],
      isActive: false,
    };
  }

  getTabInput1(id: keyof Content) {
    return this.content?.[id];
  }

  reload(event: boolean) {
    if (event) this.ngOnInit();
  }

  ngOnInit(): void {
    this.getProfile();
    this.so.getObservable().subscribe((isSaveAction) => {
      console.log('save action::> ', isSaveAction);
      this.save.emit(true);
      if (isSaveAction) this.getProfile();
    });
  }

  quickEdit() {
    this.dialog
      .open(CandidateQuickEditComponent, {
        maxWidth: this.mobileView ? '500px' : '100%',
        width: this.mobileView ? '100%' : '50%',
        ...(this.mobileView && { height: 'auto', position: { bottom: '0%' } }),
        closeOnNavigation: true,
        disableClose: true,
        ...(this.mobileView ? {} : { panelClass: 'dialog-responsive' }),
      })
      .afterClosed()
      .subscribe(() => {
        this.getProfile();
      });
  }

  getProfile() {
    if (this.userId) {
      this.profileService.getNewProfile(this.userId).then((res: any) => {
        console.log(res);
        const id = res?.['smartieUser'].id;
        this.profile = res?.['smartieUser'];

        this.profile.fullName = this.profile.fullName
          ? this.profile.fullName
          : this.profile.email.split('@')[0];
        this.workExperienceDetails = {
          id,
          ...res?.['workExperienceDetails'],
        };
        this.content = {
          profile: res?.['smartieUser'],
          personalInfo: {
            addressDetails: res?.['personalInformation']?.addressDetails,
            maritalStatus: res?.['smartieUser'].maritalStatus,
            id,
            ...res?.['personalInformation'],
          },
          idProof: {
            id,
            idProof: res?.['idProofDetails'],
            firstName: this.profile.firstName,
            lastName: this.profile.lastName,
          },
          // fitment: {
          //   id,
          //   lob: res?.['workExperienceDetails']?.lob,
          //   ...res?.['fitmentDetails'],
          // },
          education: { id: id, educationDetails: res?.['educationDetails'] },
          work: { id, ...res?.['workExperienceDetails'] },
          workExp: { id, workDetails: res?.['workDetails'] },
        };

        if (this.content.work.experienceLevel === 'FRESHER') {
          this.tabContents = this.tabContents.filter((e) => {
            return e.id !== 'workExp';
          });
        } else {
          this.tabContents = this.originalList;
        }
      });
    }
  }

  onTabChange(event: MatTabChangeEvent) {
    this.router
      .navigate([], {
        relativeTo: this.activatedRoute,
        queryParams: {
          tab: event.index,
        },
        queryParamsHandling: 'merge',
      })
      .then(() => {
        this.getProfile();
      });
  }

  openFileSelector() {
    this.fileInput.nativeElement.click();
  }

  onFileSelected(event: any) {
    const file = event.target.files[0];
    console.log("MY FILE", file);
    if (file) {
      console.log('Selected file:', file);
      this.uploadProfileImage(file);
    }
  }

  uploadProfileImage(file: File) {
    // You would typically send the file to a server here
    console.log('Uploading file:', file);
  }
}


sample service api request

getProfileByID(id: any): Promise<CandidateProfileResponse> {
    const promise = new Promise<CandidateProfileResponse>((resolve, reject) => {
      this.httpClient
        .get<CandidateProfileResponse>(
          `${environment.API_URL}candidate-services/profile/${id}`
        )
        .subscribe({
          next: (res: any) => {
            resolve(res);
          },
          error: (err: HttpErrorResponse) => {
            reject(err);
          },
        });
    });
    return promise;
  }

Apis

1) upload-profile-image
' `${environment.API_URL}/candidate-services/profile/15aadc80-de9e-4f3f-a317-a6a407317d8f/image/dummy.png

2) get profile image
' `${environment.API_URL}/candidate-services/profile/15aadc80-de9e-4f3f-a317-a6a407317d8f/image' \

3) delete profile image
' `${environment.API_URL}/candidate-services/profile/15aadc80-de9e-4f3f-a317-a6a407317d8f/image' \
