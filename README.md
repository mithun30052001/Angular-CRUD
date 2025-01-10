https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

Add a label or text near the upload button that clearly states the acceptable file formats (e.g., PNG, JPEG, PDF) and the maximum file size limit (e.g., 5MB).
 <div class="col-lg-4 col-sm-4" *ngIf="isHiringUpdate === true && openNumbersChanged">
        <div class="form-group ags-form-group">
          <label for="openNumbersProof" class="form-label">
            Upload Proof<span class="required"></span>
          </label>
          <input
            type="file"
            formControlName="openNumbersProof"
            class="form-control"
            accept=".pdf,.png,.jpg,.jpeg"
            (change)="onFileChange($event)"
            required
          />
        </div>
      </div>


.css

.form-label .file-info {
  font-size: 0.875rem; /* Slightly smaller text */
  color: #6c757d; /* Muted color for the file info */
  display: block; /* Ensures the text appears on a new line */
  margin-top: 5px; /* Adds some space above the file information */
}

.form-label .file-info strong {
  color: #495057; /* Darker color for the important information */
}




    <div class="col-lg-4 col-sm-4" *ngIf="isHiringUpdate === true && openNumbersChanged">
  <div class="form-group ags-form-group">
    <label for="openNumbersProof" class="form-label">
      Upload Proof <span class="required"></span><br />
      <small class="file-info">
        Acceptable file formats: <strong>PDF, PNG, JPEG, JPG</strong> | Maximum file size: <strong>1 MB</strong>
      </small>
    </label>
    <input
      type="file"
      formControlName="openNumbersProof"
      class="form-control"
      accept=".pdf,.png,.jpg,.jpeg"
      (change)="onFileChange($event)"
      required
    />
  </div>
</div>
