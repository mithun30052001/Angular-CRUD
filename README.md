https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

panel-schedule-dialog.component.html

<div class="upload-candidate-modal">
  <div class="upload-candidate-header">
    <h5 class="heading-text">
      {{ dialogHeading }}
    </h5>
    <div>
      <button class="close-btn" (click)="closeModal()">
        <app-icon class="app-icon" icon="close"></app-icon>
      </button>

      <!-- <img
        aria-hidden="true"
        src="assets/images/close-icon.svg"
        alt=""
        (click)="closeModal()"
      /> -->
    </div>
  </div>
  <form [formGroup]="panelForm" (ngSubmit)="onSubmit()">
    <div class="upload-candidate-body">
      <div class="row">
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Job<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select placeholder="Select Job" formControlName="jobId">
                <mat-option
                  [title]="job.jobTitle"
                  *ngFor="let job of jobs"
                  [value]="job.id"
                  >{{ job.display }}</mat-option
                >
              </mat-select> -->
              <input
                [formControl]="jobAcControl"
                type="text"
                placeholder="search jobs"
                aria-label="string"
                matInput
                [matAutocomplete]="jobAutoComplete"
              />
              <mat-autocomplete
                [displayWith]="displayJobFn"
                autoActiveFirstOption
                #jobAutoComplete="matAutocomplete"
                (optionSelected)="jobSelected($event)"
              >
                <mat-option
                  *ngFor="let item of jobAc$ | async; let index = index"
                  [value]="item"
                >
                  {{ item.display | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Requisition Number<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select placeholder="Select Job" formControlName="jobId">
                <mat-option
                  [title]="job.jobTitle"
                  *ngFor="let job of jobs"
                  [value]="job.id"
                  >{{ job.display }}</mat-option
                >
              </mat-select> -->
              <input
                [formControl]="reqAcControl"
                type="text"
                placeholder="select requisition number"
                aria-label="string"
                matInput
                [matAutocomplete]="reqAutoComplete"
              />
              <mat-autocomplete
                autoActiveFirstOption
                #reqAutoComplete="matAutocomplete"
                (optionSelected)="reqSelected($event)"
              >
                <mat-option
                  *ngFor="let item of reqAc$"
                  [value]="item.requisitionId"
                >
                   {{ item.requisitionId }} ({{ item.clientName }})
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="assignor" class="form-label"
              >Candidates<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select
                placeholder="Select Candidates"
                formControlName="applicationIds"
                multiple
              >
                <mat-option
                  [title]="application.email"
                  *ngFor="let application of applicationList"
                  [value]="application.applicationId"
                  >{{ application.fullName }}{{application.mobile}}</mat-option
                >
              </mat-select> -->
              <mat-chip-grid
                #candidateChipGrid
                aria-label="Candidate selection"
              >
                <mat-chip-row
                  *ngFor="let item of selectedCandidates"
                  (removed)="removec(item.id)"
                >
                  {{ item.name }} ({{ item.mobile }})
                  <button matChipRemove [attr.aria-label]="'remove ' + item">
                    <app-icon icon="close_small"></app-icon>
                  </button>
                </mat-chip-row>
              </mat-chip-grid>
              <input
                placeholder="Search candidates"
                #candidateAutoCompleteInput
                [formControl]="applicationAcControl"
                [matChipInputFor]="candidateChipGrid"
                [matAutocomplete]="candidateAuto"
              />
              <mat-autocomplete
                [displayWith]="displayFullNameFn"
                #candidateAuto="matAutocomplete"
                (optionSelected)="candidateSelected($event)"
              >
                <mat-option
                  *ngFor="let application of applicationAc$ | async"
                  [value]="application"
                >
                  <!-- <p *ngIf="!checkIfDuplicate(panel)">
                    {{ panel.employeeName }}
                  </p> -->
                  {{ application.name }} ({{ application.mobile }})
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-inner">
            <label class="form-label" for="employeeId">
              Panel member<span class="required"></span
            ></label>
            <mat-form-field>
              <!-- <mat-select
                (selectionChange)="onSelect($event, true)"
                placeholder="Select panel member"
                name="employeeId"
                multiple="false"
                formControlName="employeeId"
              >
                <mat-option
                  [title]="employee.emailId"
                  [value]="employee.employeeId"
                  *ngFor="let employee of panelMembers"
                  >{{ employee.employeeName }}</mat-option
                >
              </mat-select> -->

              <input
                [formControl]="primaryPanelAcControl"
                type="text"
                placeholder="search primary panel member"
                aria-label="string"
                matInput
                [matAutocomplete]="primaryAutoComplete"
              />
              <mat-autocomplete
                [displayWith]="displayEmailFn"
                autoActiveFirstOption
                #primaryAutoComplete="matAutocomplete"
                (optionSelected)="priSelected($event)"
              >
                <mat-option
                  *ngFor="
                    let item of primaryPanelAc$ | async;
                    let index = index
                  "
                  [value]="item"
                >
                  {{ item.employeeName | titlecase }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="secondaryEmployeeIds" class="form-label"
              >Secondary panel members</label
            >
            <mat-form-field appearance="outline">
              <mat-chip-grid #chipGrid aria-label="Secondary panel selection">
                <mat-chip-row
                  *ngFor="let fruit of selectedPanel"
                  (removed)="remove(fruit.employeeId)"
                >
                  {{ fruit.employeeName }}
                  <button matChipRemove [attr.aria-label]="'remove ' + fruit">
                    <app-icon icon="close_small"></app-icon>
                  </button>
                </mat-chip-row>
              </mat-chip-grid>
              <input
                placeholder="Search secondary panel members"
                #secAutoCompleteInput
                [formControl]="secondaryPanelAcControl"
                [matChipInputFor]="chipGrid"
                [matAutocomplete]="secondaryPanelAuto"
              />
              <mat-autocomplete
                [displayWith]="displayEmailFn"
                #secondaryPanelAuto="matAutocomplete"
                (optionSelected)="secSelected($event)"
              >
                <mat-option
                  *ngFor="let panel of secondaryPanelAc$ | async"
                  [value]="panel"
                >
                  <!-- <p *ngIf="!checkIfDuplicate(panel)">
                    {{ panel.employeeName }}
                  </p> -->
                  {{ panel.employeeName }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>

            <!-- <mat-form-field>
              <mat-select
                (selectionChange)="onSelect($event, false)"
                name="secondaryEmployeeIds"
                placeholder="Select Candidates"
                formControlName="secondaryEmployeeIds"
                multiple
              >
                <mat-option
                  [title]="employee.emailId"
                  [value]="employee.employeeId"
                  *ngFor="let employee of secondaryPanelMembers"
                  >{{ employee.employeeName }}</mat-option
                >
              </mat-select>
            </mat-form-field> -->
          </div>
        </div>
        <div class="col-lg-12 col-sm-12">
          <div class="form-group ags-form-group">
            <label for="email" class="form-label"
              >Time Zone<span class="required"></span
            ></label>
            <mat-form-field>
              <input
                type="text"
                placeholder="Select Timezone"
                matInput
                [matAutocomplete]="auto"
                formControlName="timezone"
              />
              <mat-autocomplete
                #auto="matAutocomplete"
                [displayWith]="displayFn"
                name="timezone"
                (optionSelected)="timeZoneChange($event)"
              >
                <mat-option
                  class="time-select-panel"
                  *ngFor="let item of filteredTimeZoneList | async"
                  [value]="item"
                >
                  {{
                    '(UTC' +
                      item.currentTimeFormat.substring(0, 6) +
                      ') - ' +
                      item.mainCities.join(', ')
                  }}
                </mat-option>
              </mat-autocomplete>
            </mat-form-field>
          </div>
        </div>
      </div>
      <div class="row d-flex align-items-center">
        <div class="col-lg-6 col-sm-12">
          <div class="form-group form-inner">
            <label for="email" class="form-label"
              >Date<span class="required"></span
            ></label>
            <div class="form-datepicker">
              <input
                (dateChange)="onDateChange($event)"
                [min]="min"
                placeholder="Select Date"
                formControlName="date"
                readonly
                class="form-control"
                [matDatepicker]="picker"
              />
              <mat-datepicker-toggle
                matIconSuffix
                [for]="picker"
              ></mat-datepicker-toggle>
            </div>
            <mat-datepicker #picker></mat-datepicker>
            <div
              class="form-invalid"
              *ngIf="timingList.length < 1 && isAvailableTimeChecked"
            >
              <p>Not available for the given date</p>
            </div>
          </div>
        </div>
        <div class="col-lg-6 col-sm-12">
          <div class="">
            <div class="form-inner">
              <label class="form-label" for="employeeId">
                Allocate schedule<span class="required"></span
              ></label>
              <mat-form-field>
                <mat-select
                  placeholder="Schedule"
                  name="employeeId"
                  multiple="false"
                  formControlName="timings"
                >
                  <mat-option *ngFor="let option of timingList" [value]="option"
                    ><span class="mat-radio-label"
                      >{{ option['startTime'] | date : 'hh:mm a' }} -
                      {{ option['endTime'] | date : 'hh:mm a' }}</span
                    ></mat-option
                  >
                </mat-select>
              </mat-form-field>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="upload-candidate-footer">
      <div>
        <button
          title="Cancel"
          mat-dialog-close
          class="ags-outline-btn ags-hmd44 btn-font16 ags-padding1624"
        >
          Cancel
        </button>
      </div>
      <div>
        <button
          [disabled]="panelForm.invalid"
          title="Submit"
          type="submit"
          class="ags-primary-btn ags-hmd44 btn-font16 ags-padding1624"
        >
          {{scheduleButtonText}}
        </button>
      </div>
    </div>
  </form>
</div>
