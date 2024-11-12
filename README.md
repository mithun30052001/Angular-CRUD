populateFormWithEventData() {
    if (this.eventData) {
      const { panelMembers, candidates, start, end, title } = this.eventData;

      this.panelForm.patchValue({
        date: moment(start).format('YYYY-MM-DD'),
        timings: {
          startTime: moment(start).format('HH:mm'),
          endTime: moment(end).format('HH:mm'),
        },
        timezone: "Asia/Kolkata",
      });

      if (candidates && candidates.length > 0) {
        this.selectedCandidates = candidates;
        this.applicationAcControl.setValue(
          candidates.map((candidate: any) => candidate.applicationId)
        );
      }

      if (panelMembers && panelMembers.length > 0) {
        const primaryMember = panelMembers.find(
          (member: any) => member.primaryPanelMember
        );
        const secondaryMembers = panelMembers.find(
          (member: any) => !member.primaryPanelMember
        );

        this.primaryPanelAcControl.setValue(primaryMember);
        this.secondaryPanelAcControl.setValue(
          secondaryMembers.map((member: any) => member.employeeId)
        );
      }

      if (title) {
        this.jobAcControl.setValue(title);
        this.panelForm.patchValue({ jobId: title });
      }
    }
  }
