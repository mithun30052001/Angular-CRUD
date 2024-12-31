https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

panel-schedule.component.ts

reqAcControl = new FormControl();
  reqAc$: Observable<any[]>;  // The list of requisition numbers
  filteredOptions$: Observable<any[]>;

  ngOnInit() {
    // Assuming `reqAc$` is populated from an API or service
    this.filteredOptions$ = this.reqAcControl.valueChanges.pipe(
      debounceTime(300),
      switchMap(value => this.filterOptions(value))
    );
  }

  filterOptions(value: string): Observable<any[]> {
    if (!value) {
      return [];
    }

    return this.reqAc$.pipe(
      map(options => 
        options.filter(option => option.requisitionId.includes(value))
      )
    );
  }

  reqSelected(event: any): void {
    const selectedValue = event.option.value;
    if (!this.reqAc$.some(item => item.requisitionId === selectedValue)) {
      this.reqAcControl.setValue('');  // Clear invalid value
    }
  }


panel-schedule.component.html

<div class="col-lg-12 col-sm-12">
  <div class="form-group ags-form-group">
    <label for="assignor" class="form-label">
      Requisition Number<span class="required"></span>
    </label>
    <mat-form-field>
      <input
        [formControl]="reqAcControl"
        type="text"
        placeholder="select requisition number"
        matInput
        [matAutocomplete]="reqAutoComplete"
      />
      <mat-autocomplete
        autoActiveFirstOption
        #reqAutoComplete="matAutocomplete"
        (optionSelected)="reqSelected($event)"
      >
        <mat-option *ngFor="let item of filteredOptions$ | async" [value]="item.requisitionId">
          {{ item.requisitionId }} ({{ item.clientName }})
        </mat-option>
      </mat-autocomplete>
    </mat-form-field>
  </div>
</div>
