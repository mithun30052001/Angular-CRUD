https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

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
