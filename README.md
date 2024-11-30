https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

viewRemarkImage(remarkId: string): void {
    this.reqService.getRemarkImage(remarkId).subscribe(
      (imageData) => {
        // Handle image display logic, for example, open in a new window or modal
        const imageUrl = URL.createObjectURL(imageData);
        window.open(imageUrl, '_blank');
      },
      (error) => {
        console.error('Error fetching remark image:', error);
      }
    );
  }


{
    "message": "SUCCESS",
    "messageDesc": "https://agstadtdev.s3.ap-south-1.amazonaws.com/0f84f74c-c9e5-4393-9219-074a2426c6b9_Architecture.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20241130T075553Z&X-Amz-SignedHeaders=host&X-Amz-Expires=604800&X-Amz-Credential=AKIASP5FLSULKIASX4FA%2F20241130%2Fap-south-1%2Fs3%2Faws4_request&X-Amz-Signature=939f866bdc5eab042c99e63ca35d3170a5b451cdc95ab114fea04506603d6dda",
    "fileName": "0f84f74c-c9e5-4393-9219-074a2426c6b9_Architecture.jpg",
    "responseStatus": 200
}


viewRemarkImage(remarkId: string): void {
  this.reqService.getRemarkImage(remarkId).subscribe(
    (response) => {
      // Ensure the response contains the signed URL in 'messageDesc'
      const imageUrl = response.messageDesc;
      const fileName = response.fileName;  // You can use this as the downloaded file name

      // Fetch the image data as a Blob using the signed URL
      fetch(imageUrl)
        .then((res) => res.blob())  // Convert the response to a Blob
        .then((blob) => {
          // Create a temporary link element for the download
          const a = document.createElement('a');
          const objectUrl = URL.createObjectURL(blob);  // Create a URL for the Blob
          
          a.href = objectUrl;
          a.download = fileName || 'download.jpg';  // Use fileName from response or a default name
          
          // Programmatically trigger the click to download the file
          a.click();
          
          // Clean up the object URL after the download starts
          URL.revokeObjectURL(objectUrl);
        })
        .catch((error) => {
          console.error('Error fetching the image:', error);
        });
    },
    (error) => {
      console.error('Error fetching remark image data:', error);
    }
  );
}

<span [title]="element.imageName">
  {{ element.imageName | slice:0:20 }}{{ element.imageName.length > 20 ? '...' : '' }}
</span>
