https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

<div *ngIf="['ASSESMENT', 'VERSANT', 'PANEL'].includes(interview.interviewType)" class="assessment-interview">
        <form [formGroup]="form" (ngSubmit)="onSubmit()">
          <div class="d-flex justify-content-start">
            <div class="mt-3 form-group form-inner">
              <p class="assment-subheading">
                Status:&nbsp;&nbsp;
                <mat-form-field class="example-form-field">
                  <mat-select placeholder="Select status" formControlName="status">
                    <mat-option
                      [value]="option.value"
                      *ngFor="let option of PANEL_STATUS_OPTIONS"
                    >
                      {{ option.display }}
                    </mat-option>
                  </mat-select>
                </mat-form-field>
              </p>
            </div>
          </div>
          
          <div class="form-inner mt-3">
            <label class="panel-feedback" for="feedBack">Panel remarks</label>
            <textarea
              formControlName="feedBack"
              placeholder="Panel Feedback"
              class="text-area"
            ></textarea>
          </div>
          
          <div class="submit-btn">
            <button mat-raised-button color="primary" type="submit" [disabled]="form.invalid">Submit</button>
          </div>
        </form>
      </div>
