https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d


validateReqAc(value: string): void {
    const isValid = this.reqAc$.some((item: { requisitionId: string }) => item.requisitionId === value);

    if (!isValid) {
      this.reqAcControl.setValue(''); // Clear the value if it's not valid
      this.reqAcControl.markAsTouched(); // Mark as touched to show validation error
    }
  }

  /**
   * Handler when an option is selected from the dropdown
   */
  reqSelected(event: MatAutocompleteSelectedEvent): void {
    this.panelForm.get('requisitionId')?.setValue(event.option.value); // Set the form value
  }

  /**
   * Input change handler to perform any additional checks or updates
   */
  onInputChange(event: any): void {
    const value = event.target.value;
    this.validateReqAc(value);
  }





 <mat-select 
        placeholder="Select Requisition Number" 
        formControlName="reqAcControl"
        (selectionChange)="reqSelected($event)"
      >
        <mat-option
          *ngFor="let item of reqAc$"
          [value]="item.requisitionId"
        >
          {{ item.requisitionId }} ({{ item.clientName }})
        </mat-option>
      </mat-select>
