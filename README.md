add-panel-member-component.ts
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
    console.log("ADD PANEL MEMBER FORM",this.panelMemberForm.value);
    var payload = {
      employeeId: this.panelMemberForm.value.panelMemberId,
      type: 'ADD_PANELMEMBER',
      primary: this.panelMemberForm.value.makePrimary == '0' ? false : true,
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


add-panel-member.component.html
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

