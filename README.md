https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

<div class="app-icon-overlay" *ngIf="isHovered" (mouseover)="isHovered=true" (mouseleave)="isHovered=false">
        <button class="event-btn" (click)="triggerFileInput()">
          <app-icon icon="upload_ic" title="Upload Profile" />
        </button>
      </div>

      <!-- Hidden File Input -->
      <input type="file" #fileInput style="display: none" (change)="onFileSelected($event)" />

profile.scss

.profile-avator {
  position: relative;
  width: 100px; /* Adjust as needed */
  height: 100px; /* Adjust as needed */
  border-radius: 50%;
  overflow: hidden;
}

.app-icon-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background: rgba(0, 0, 0, 0.5); /* Semi-transparent background */
  opacity: 0;
  transition: opacity 0.3s ease-in-out;
}

.profile-avator:hover .app-icon-overlay {
  opacity: 1;
}

.event-btn {
  background-color: transparent;
  border: none;
  cursor: pointer;
}

.event-btn app-icon {
  color: white;
  font-size: 24px;
}

profile.component.ts

import { Component, EventEmitter, Input, OnInit, Output, ViewChild } from '@angular/core';
import { MatDialog } from '@angular/material/dialog';
import { MatDialogConfig } from '@angular/material/dialog';

@Component({
  selector: 'app-profile',
  templateUrl: './profile.component.html',
  styleUrls: ['./profile.component.scss'],
})
export class ProfileComponent implements OnInit {
  @ViewChild('fileInput', { static: false }) fileInput: any;
  
  profileImage: File | null = null;
  imagePreviewUrl: string | ArrayBuffer | null = null;
  isHovered: boolean = false;

  constructor(private dialog: MatDialog) {}

  ngOnInit(): void {
    // Initial setup logic if needed
  }

  // Trigger the file input when the upload icon is clicked
  triggerFileInput() {
    this.fileInput.nativeElement.click();
  }

  // Handle file selection
  onFileSelected(event: Event) {
    const input = event.target as HTMLInputElement;
    if (input?.files?.[0]) {
      this.profileImage = input.files[0];
      const reader = new FileReader();
      reader.onload = () => {
        this.imagePreviewUrl = reader.result;
        this.openImagePreviewModal();
      };
      reader.readAsDataURL(this.profileImage);
    }
  }

  // Open modal to preview and upload image
  openImagePreviewModal() {
    const dialogConfig = new MatDialogConfig();
    dialogConfig.data = {
      imageUrl: this.imagePreviewUrl,
      onRemove: this.removeImage.bind(this),
      onUpload: this.uploadImage.bind(this),
    };
    const dialogRef = this.dialog.open(ImagePreviewDialogComponent, dialogConfig);
  }

  // Remove the selected image
  removeImage() {
    this.profileImage = null;
    this.imagePreviewUrl = null;
  }

  // Upload the selected image (you can add your upload logic here)
  uploadImage() {
    if (this.profileImage) {
      // Logic to upload the image to server or handle it as needed
      console.log('Uploading image:', this.profileImage);
    }
  }
}




app-icon
<button class="event-btn" (click)="openEditEvent()">
    <app-icon icon="edit_ic"></app-icon>
</button>

profile.component.ts

import {
  Component,
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
}



profile-component.html profile image div

<div class="row d-flex align-items-center">
      <div class="col-sm-4 col-lg-2 col-3">
        <div class="profile-avator">
          <img src="assets/images/profile-pic.png" alt="" class="img-fluid" />
         
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

