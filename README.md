https://teams.microsoft.com/l/meetup-join/19%3ameeting_ODliN2VjZjktZjM2MS00OGQ4LWFhMzUtZjAwNTJkMTRkY2Y4%40thread.v2/0?context=%7b%22Tid%22%3a%22f6fb95f2-bd20-41a4-b19a-c7fcf96d09a7%22%2c%22Oid%22%3a%2238c62280-1dc6-4ce5-b5b4-8a068650cb44%22%7d

export interface Job {
  id: string;
  jobTitle: string;
  oracleJobReference: string;
  location: string;
  shift: string;
  shortDesc: string;
  longDesc: string;
  openPositions: number;
  postedAt: string;
  visibility: boolean;
  jobType: string;
  processType: string;
  jobHours: string;
  experienceLevel: string;
  cashBonus: number;
  education: string | null;
}

export interface PeriodicElement {
  id: string;
  createdDate: string;
  updatedDate: string;
  active: boolean;
  createdBy: string;
  updatedBy: string;
  job: Job;
  remarks: string; // Assuming remarks is part of the response
}


const data = response.response;
        this.dataSource = data.map((item: any) => ({
          id: item.id,
          createdDate: item.createdDate,
          updatedDate: item.updatedDate,
          active: item.active,
          createdBy: item.createdBy,
          updatedBy: item.updatedBy,
          job: {
            id: item.job.id,
            jobTitle: item.job.jobTitle,
            oracleJobReference: item.job.oracleJobReference,
            location: item.job.location,
            shift: item.job.shift,
            shortDesc: item.job.shortDesc,
            longDesc: item.job.longDesc,
            openPositions: item.job.openPositions,
            postedAt: item.job.postedAt,
            visibility: item.job.visibility,
            jobType: item.job.jobType,
            processType: item.job.processType,
            jobHours: item.job.jobHours,
            experienceLevel: item.job.experienceLevel,
            cashBonus: item.job.cashBonus,
            education: item.job.education,
          },
          remarks: '', // Add logic to extract remarks if available
        }));
