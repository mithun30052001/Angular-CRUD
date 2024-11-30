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
