https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

app-icon
<button class="event-btn" (click)="openEditEvent()">
    <app-icon icon="edit_ic"></app-icon>
</button>

Profile div in header-component.html
 <div class="col-sm-4 col-md-4 col-lg-3">
            <div class="profile float-end">
              <button
                title="Account"
                mat-button
                [matMenuTriggerFor]="belowMenu"
                (click)="getProfile()"
                class="profile-btn border-0"
              >
                <div class="refer-buddy-section">
                  <div>
                    <img
                      class="profile-btn-image"
                      src="assets/images/profile-pic.png"
                      alt=""
                      class="img-fluid"
                    />
                  </div>
                  <div class="profile-text">Account</div>
                </div>
              </button>
              <mat-menu #belowMenu="matMenu" yPosition="below" class="myMenu">
                <div
                  aria-hidden="true"
                  class="logout-menu"
                  (click)="$event.stopPropagation()"
                >
                  <div class="avator-size">
                    <img src="assets/images/profile-pic.png" alt="" />
                  </div>
                  <h5>{{ profileRecord.fullName }}</h5>
                  <p>{{ profileRecord.email }}</p>
                  <button
                    title="Sign Out"
                    mat-raised-button
                    class="top-gap-1 logout-button"
                    (click)="logout()"
                  >
                    <div class="logout-text">
                      <app-icon icon="sign_out"></app-icon>
                      <div class="signOut-text">Sign Out</div>
                    </div>
                  </button>
                </div>
              </mat-menu>
            </div>
          </div>

header-component.ts

import { AuthService } from '@/src/app/shared/services/auth.service';
import { ProfileService } from '@/src/app/shared/services/profile.service';
import {
  checkCandidateAccess,
  checkEmployeeAccess,
  checkHrAccess,
  checkVendorAccess,
  checkPanelAccess,
} from '@/src/app/shared/utils/utils';
import { MediaMatcher } from '@angular/cdk/layout';
import { Component, Input, OnInit } from '@angular/core';
import { MatDialog } from '@angular/material/dialog';
import { MatMenuTrigger } from '@angular/material/menu';
import { CandidateReferralPolicyModalComponent } from '../../../candidate-referal-policy-modal/candidate-referral-policy-modal.component';
import { EmployeeReferralPolicyComponent } from '../../../employee-referal-policy/employee-referral-policy.component';
import { ReferBuddyComponent } from '../../../refer-buddy/refer-buddy.component';
import { ResponsiveViewComponent } from '@/src/app/shared/components/responsive-view/responsive-view.component';
import { UploadCandidateDialogComponent } from '@/src/app/vendor/components/upload-candidate-dialog/upload-candidate-dialog.component';
import { AddCandidateDialogComponent } from '@/src/app/shared/components/add-candidate-dialog/add-candidate-dialog.component';
import { Router } from '@angular/router';
import { UtilService } from '@/src/app/shared/services/util.service';

@Component({
  selector: 'app-header',
  templateUrl: './header.component.html',
  styleUrls: ['./header.component.scss'],
})
export class HeaderComponent extends ResponsiveViewComponent implements OnInit {
  @Input() menu: MatMenuTrigger | any;
  profileRecord: any = {};
  checkHrAccess = checkHrAccess;
  checkEmployeeAccess = checkEmployeeAccess;
  checkCandidateAccess = checkCandidateAccess;
  checkVendorAccess = checkVendorAccess;
  checkPanelAccess = checkPanelAccess;
  userID = '';
  isAdmin = false;
  constructor(
    private profileService: ProfileService,
    public dialog: MatDialog,
    media: MediaMatcher,
    private authService: AuthService,
    private utilService: UtilService,
    private router: Router
  ) {
    super(media);
  }

  ngOnInit(): void {
    this.userID = localStorage.getItem('id') || '';
    this.isAdmin = this.utilService.checkAdminAccess();
    this.getProfile();
  }

  getProfile() {
    if (!this.profileRecord || !Object.keys(this.profileRecord)?.length) {
      this.profileService.getProfileByID(this.userID).then((res) => {
        this.profileRecord = res;
      });
    }
  }

  uploadCandidate() {
    this.dialog
      .open(UploadCandidateDialogComponent, {
        maxWidth: '796px',
        width: '95%',
        height: '90%',
        position: { top: '1%' },
        closeOnNavigation: true,
        disableClose: true,
        autoFocus: false,
      })
      .afterClosed()
      .subscribe((_res) => {
        if (_res) console.log(_res);
      });
  }

  addCandidate() {
    this.dialog
      .open(AddCandidateDialogComponent, {
        maxWidth: this.mobileView ? '500px' : '100%',
        height: 'auto',
        width: this.mobileView ? '100%' : '60%',
        ...(this.mobileView && {
          height: 'auto',
          position: { bottom: '0' },
        }),
        closeOnNavigation: true,
        disableClose: true,
      })
      .afterClosed()
      .subscribe((res) => {
        if (res) {
          window.location.reload();
        }
      });
  }

  logout() {
    this.authService.logout();
  }

  referBuddy() {
    this.dialog.open(ReferBuddyComponent, {
      maxWidth: this.mobileView ? '500px' : '850px',
      width: '100%',
      ...(this.mobileView && { height: 'auto', position: { bottom: '0' } }),
      closeOnNavigation: true,
      disableClose: true,
    });
  }

  referralPolicy() {
    if (checkEmployeeAccess()) {
      this.dialog.open(EmployeeReferralPolicyComponent, {
        maxWidth: this.mobileView ? '500px' : '1117px',
        width: '100%',
        height: this.mobileView ? '450px' : '700px',
        position: { bottom: '0' },
        closeOnNavigation: true,
        disableClose: true,
        data: this.mobileView ? 'referralPolicyMobile' : 'referralPolicyWeb',
      });
    } else {
      if (checkCandidateAccess()) {
        this.dialog.open(CandidateReferralPolicyModalComponent, {
          maxWidth: this.mobileView ? '500px' : '1117px',
          width: '100%',
          height: this.mobileView ? '450px' : '700px',
          position: { bottom: '0' },
          closeOnNavigation: true,
          disableClose: true,
          data: this.mobileView ? 'referralPolicyMobile' : 'referralPolicyWeb',
        });
      }
    }
  }
}
