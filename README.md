The below is open edit event in angular which calls add panel member dialog box with event data.Now fill in the panelMembers(primary and secondary from  "primaryPanelMember" value true/false),candidates,Date(from start and end value), scheduleMeeting(from start timing to end timing) and if data contains eventId change the dialog heading to Edit Panel Schedule and schedule button to Reschedule

event data

{
    "id": "f6675c2d-727c-47d9-be13-6c3c3498c477",
    "start": "2024-11-19T01:00:00.000Z",
    "end": "2024-11-19T01:30:00.000Z",
    "panelMembers": [
        {
            "id": "bacfa10c-6289-4a63-b3ac-92365b34358c",
            "primaryPanelMember": true,
            "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
            "name": "Kabildev Chellamuthu",
            "email": "kabildev.chellamuthu@agshealth.com",
            "mobile": "9600795023"
        },
          {
            "id": "bacfa10c-6289-4a63-b3ac-92365b34358c",
            "primaryPanelMember": false,
            "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
            "name": "Vishnu",
            "email": "kabildev.chellamuthu@agshealth.com",
            "mobile": "9600795023"
        }
    ],
    "candidates": [
        {
            "id": "77e98dba-6132-47b9-94d8-cedd35b16644",
            "candidateId": "b9e4375f-1816-4040-8634-6395365ca32b",
            "name": "Abc ktik",
            "email": "abc@gmail.com",
            "mobile": "9876543210",
            "applicationId": "878e5247-5a7d-4af6-bf75-2bb8eff07137",
            "applicationStatus": null,
            "interviewStatus": "PROGRESS",
            "active": true,
            "feedback": null,
            "panelUpdatedEmployee": null
        }
    ],
    "title": "Interview | Voice Experienced Hb",
    "color": {
        "primary": "#2b3874",
        "secondary": "#fafaff"
    },
    "actions": [
        {
            "label": "https://teams.microsoft.com/l/meetup-join/19%3ameeting_MTcwMjNmNjItODE3NC00NTJhLWFlZDQtM2E4N2VlNThlOGE2%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%229c344eb3-53a2-45af-9f02-f764bc2f2e49%22%7d"
        }
    ],
    "allDay": false,
    "resizable": {
        "beforeStart": false,
        "afterEnd": false
    },
    "draggable": false,
    "meta": {
        "meetingLink": "https://teams.microsoft.com/l/meetup-join/19%3ameeting_MTcwMjNmNjItODE3NC00NTJhLWFlZDQtM2E4N2VlNThlOGE2%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%229c344eb3-53a2-45af-9f02-f764bc2f2e49%22%7d",
        "interviewType": null,
        "hrId": "0002458d-b8b9-49a9-b5a1-e9598328460b",
        "panelMembers": [
            {
                "id": "bacfa10c-6289-4a63-b3ac-92365b34358c",
                "primaryPanelMember": true,
                "employeeId": "2f9f1f2d-977b-4acf-80e6-9ebaef57e030",
                "name": "Kabildev Chellamuthu",
                "email": "kabildev.chellamuthu@agshealth.com",
                "mobile": "9600795023"
            }
        ],
        "candidates": [
            {
                "id": "77e98dba-6132-47b9-94d8-cedd35b16644",
                "candidateId": "b9e4375f-1816-4040-8634-6395365ca32b",
                "name": "Abc ktik",
                "email": "abc@gmail.com",
                "mobile": "9876543210",
                "applicationId": "878e5247-5a7d-4af6-bf75-2bb8eff07137",
                "applicationStatus": null,
                "interviewStatus": "PROGRESS",
                "active": true,
                "feedback": null,
                "panelUpdatedEmployee": null
            }
        ],
        "eventId": "AAMkADAzOTM5MDM5LWU1NDUtNDcxZS04NGNkLTQ5OTI4YmNhZjNiNQBGAAAAAADclWry2I3nSqkjw7GWRnvVBwA1uIZfWZT2QrlhZYeipAE7AAAAAAENAAA1uIZfWZT2QrlhZYeipAE7AAC4bgsSAAA="
    }
}

openEditEvent method 

openEditEvent() {
    console.log("Data inside event", this.event);
    this.dialog.open(PanelScheduleDialogComponent, {
      maxWidth: this.mobileView ? '100%' : '100%',
      width: this.mobileView ? '100%' : '654px',
      height: 'auto',
      ...(this.mobileView && { height: 'auto', position: { bottom: '0' } }),
      closeOnNavigation: true,
      disableClose: false,
      autoFocus: false,
      panelClass: 'custom-dialog-container',
      data: this.event,
    });
  }
  declineEvent(eventId: any) {}
}

add-panel-member-dialog.ts

import { Component, Inject } from '@angular/core';
import { Form, FormBuilder, FormGroup, Validators } from '@angular/forms';
import {
  MAT_DIALOG_DATA,
  MatDialog,
  MatDialogRef,
} from '@angular/material/dialog';
import { ApplicationService } from '../../../services/job-application.service';
import { CalendarService } from '../../../services/calendar.service';
import { MessageModalsComponent } from '@/src/app/common-modules/message-modals/message-modals.component';

@Component({
  selector: 'app-add-panel-member-dialog',
  templateUrl: './add-panel-member-dialog.component.html',
  styleUrls: ['./add-panel-member-dialog.component.scss'],
})
export class AddPanelMemberDialogComponent {
  employeeList: any;
  panelMemberForm: FormGroup;

  constructor(
    private dialogRef: MatDialogRef<AddPanelMemberDialogComponent>,
    public dialog: MatDialog,
    private formBuilder: FormBuilder,
    @Inject(MAT_DIALOG_DATA) public data: string,
    private applicationService: ApplicationService,
    private calendarService: CalendarService
  ) {
    this.panelMemberForm = this.formBuilder.group({
      panelMember: ['', Validators.required],
      panelMemberId: [''],
      makePrimary: ['0', Validators.required],
      remarks: [''],
    });
    this.getEmployeeList('');
  }
  closeModal() {
    this.dialogRef.close();
  }
  onEmployeeSearch($event: any) {
    this.getEmployeeList($event.target.value);
  }

  onEmployeeSelected(data: any) {
    var item = data.option.value;
    this.panelMemberForm.get('panelMember')?.setValue(item);
    this.panelMemberForm.get('panelMemberId')?.setValue(item.employeeId);
  }

  getEmployeeList(searchString: string) {
    this.applicationService.getPanelMembersV2(searchString).subscribe((res) => {
      this.employeeList = res;
    });
  }

  async submit() {
    console.log(this.panelMemberForm);
    var payload = {
      employeeId: this.panelMemberForm.value.panelMemberId,
      type: 'ADD_PANELMEMBER',
      primary: this.panelMemberForm.value.makePrimary == '0' ? true : false,
      feedBack: this.panelMemberForm.value.remarks,
    };
    var res = await this.calendarService.addPanelMember(this.data, payload);
    const dialogRef = this.dialog.open(MessageModalsComponent, {
      maxWidth: '500px',
      width: '100%',
      height: 'auto',
      closeOnNavigation: true,
      disableClose: true,
      data: res ? 'panelMemberAddedSuccess' : 'errorOccurWeb',
    });

    dialogRef.afterClosed().subscribe((_result: any) => {
      this.dialogRef.close();
    });
  }

  displayFullNameFn(value: any) {
    return value ? value.employeeName : '';
  }
}

add-panel-member-dialog.html

<div class="add-panel-modal">
  <div class="add-panel-header">
    <h5 class="heading-text">Add Panel Members</h5>
    <app-icon icon="close" (click)="closeModal()"></app-icon>
  </div>
  <form class="form-item">
    <div class="add-panel-body">
      <div class="resume-upload">
        <div class="row">
          <div class="col-lg-12 col-sm-12">
            <div class="form-group form-inner">
              <label class="form-label" for="panelMember"
                >Panel Member<span class="required"></span
              ></label>
              <div>
                <input
                  name="panelMember"
                  type="text"
                  formControlName="panelMember"
                  placeholder="Search..."
                  class="form-control"
                  [matAutocomplete]="auto"
                  (change)="onEmployeeSearch($event)"
                />
                <mat-autocomplete
                  #auto="matAutocomplete"
                  [displayWith]="displayFullNameFn"
                  name="autocomp"
                  (optionSelected)="onEmployeeSelected($event)"
                >
                  <mat-option
                    class="time-select-panel"
                    *ngFor="let item of employeeList"
                    [value]="item"
                  >
                    {{ item.employeeName }}
                  </mat-option>
                </mat-autocomplete>
              </div>
            </div>
          </div>
          <div class="col-lg-12 col-sm-12">
            <div class="form-group form-inner">
              <label class="form-label" for="makePrimary"
                >Make as Primary <span class="required"></span
              ></label>
              <mat-radio-group
                formControlName="makePrimary"
                aria-label="Select an option"
              >
                <mat-radio-button [value]="'1'">
                  <span class="mat-radio-label">Yes</span>
                </mat-radio-button>
                <mat-radio-button [value]="'0'">
                  <span class="mat-radio-label">No</span>
                </mat-radio-button>
              </mat-radio-group>
            </div>
          </div>
          <div class="col-lg-12 col-sm-12">
            <div class="form-group form-inner">
              <label class="form-label" for="remarks">Remarks</label>
              <div>
                <textarea
                  [rows]="3"
                  name="reasonForJobChange"
                  formControlName="remarks"
                  placeholder="Panel member addition remarks"
                  class="text-area"
                ></textarea>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="add-panel-footer">
      <div class="row justify-content-end">
        <div class="col-lg-3 col-6">
          <button
            type="button"
            (click)="closeModal()"
            class="ags-outline-btn ags-hxl56 ags-padding1624 btn-font16"
          >
            Cancel
          </button>
        </div>
        <div class="col-lg-3 col-6">
          <button
            [disabled]="!panelMemberForm.valid"
            type="submit"
            class="ags-primary-btn ags-hxl56 ags-padding1624 btn-font16"
            (click)="submit()"
          >
            Save
          </button>
        </div>
      </div>
    </div>
  </form>
</div>
