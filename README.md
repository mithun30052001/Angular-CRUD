https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

{
    "message": "SUCCESS",
    "messageDesc": "https://agstadtdev.s3.ap-south-1.amazonaws.com/0c553ed6-7848-48e6-9f48-a85e809f9f77-colony-or-group-of-razorbills-alca-torda-swimming-in-the-sea-in-vicinity-of-cable-rock-at-golden-hour-large-file-size-bray-head-cowicklow-WPDNEH.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20241122T114653Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3600&X-Amz-Credential=AKIASP5FLSULKIASX4FA%2F20241122%2Fap-south-1%2Fs3%2Faws4_request&X-Amz-Signature=8f263c5f2dcac043ac0957964d374c13e5fc70566ca3deb8938f746f6628bc4c",
    "fileName": "colony-or-group-of-razorbills-alca-torda-swimming-in-the-sea-in-vicinity-of-cable-rock-at-golden-hour-large-file-size-bray-head-cowicklow-WPDNEH.jpg"
}

imageUrl

DATA URL data:application/json;base64,eyJtZXNzYWdlIjoiU1VDQ0VTUyIsIm1lc3NhZ2VEZXNjIjoiaHR0cHM6Ly9hZ3N0YWR0ZGV2LnMzLmFwLXNvdXRoLTEuYW1hem9uYXdzLmNvbS8wZDIxYTlmMy0xNDNiLTQwOTMtYjMxZS0wZjgxNTY3NGYzMTItQzAwNzM1X1RoaXJ1bmF2dWtrYXJhc3UlMjBOYXRlc2FuLmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1EYXRlPTIwMjQxMTIyVDA5NTkwMlomWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JlgtQW16LUV4cGlyZXM9MzYwMCZYLUFtei1DcmVkZW50aWFsPUFLSUFTUDVGTFNVTEtJQVNYNEZBJTJGMjAyNDExMjIlMkZhcC1zb3V0aC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotU2lnbmF0dXJlPWY4ZGQ0N2VjM2ZiMDIxNzgyNDNjZjllM2Q1NDY4Y2VkMDZmZjQ0OWI0ZTE0NTExOTk3OGZkYTNjODQ1YzViYTYiLCJmaWxlTmFtZSI6IkMwMDczNV9UaGlydW5hdnVra2FyYXN1IE5hdGVzYW4uanBnIn0=


candidate-image-preview.ts

import { Component, Inject } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialogRef } from '@angular/material/dialog';
import { ProfileService } from '@/src/app/shared/services/profile.service';

@Component({
  selector: 'app-image-preview-dialog',
  templateUrl: './candidate-image-preview-dialog.component.html',
  styleUrls: ['./candidate-image-preview-dialog.component.scss'],
})
export class CandidateImagePreviewDialogComponent {
  constructor(
    public dialogRef: MatDialogRef<CandidateImagePreviewDialogComponent>,
    @Inject(MAT_DIALOG_DATA) public data: any,
    private profileService: ProfileService
  ) {}

  // Close the dialog
  closeDialog(): void {
    this.dialogRef.close();
    }
    
    getImage(): any {
        this.profileService.getProfileImage(this.data.userId).subscribe({
          next: (response) => {
            // Create an image URL from the blob response
            const reader = new FileReader();
            reader.onloadend = () => {
              this.data.imageUrl = reader.result as string;
            };
           return reader.readAsDataURL(response);
          },
          error: (error) => {
            console.error('Error fetching profile image:', error);
          },
        });
    }

  // Delete the profile image
  deleteImage(): void {
    this.profileService.deleteProfileImage(this.data.userId).subscribe({
      next: (response) => {
        console.log('Profile image deleted successfully', response);
        this.dialogRef.close('delete'); // Close and send 'delete' signal to parent component
      },
      error: (error) => {
        console.error('Error deleting image:', error);
      },
    });
  }
}


candidate-image-preview.html

<h1 mat-dialog-title>Profile Image</h1>

<div mat-dialog-content class="text-center">
  <img [src]="getImage()" alt="Profile Image" class="img-fluid" />
</div>

<div mat-dialog-actions class="d-flex justify-content-between">
  <button mat-button (click)="closeDialog()">Close</button>
  <button mat-button (click)="deleteImage()" color="warn">Delete</button>
</div>
