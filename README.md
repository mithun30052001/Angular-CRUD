https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

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
