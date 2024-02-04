<template>
 
    <div class="card">
      <div class="">
        <DataTable v-model:selection="selectedCampaign" :value="campaigns" show-gridlines paginator :rows="15"
          sortMode="multiple" @row-select="onRowSelect" dataKey="campaignid" stripedRows tableStyle="">
          <Column selectionMode="single" headerStyle="width: 3rem"></Column>
          <Column field="campaignid" header="Campaign Id"></Column>
          <Column field="title" header="Campaign Name"></Column>
          <Column field="category" header="Campaign Category"></Column>
          <Column field="basecost" header="Budgeted"></Column>
          <Column field="startdate" header="Start Date"></Column>
          <Column field="enddate" header="End Date"></Column>
        </DataTable>
      </div>
    </div>
    <CampaignModal :visible="showModal" :campaignData="selectedCampaign" @update:visible="showModal = $event" />


</template>

<script setup>
import { onMounted, ref, inject } from "vue";
import CampaignModal from "./CampaignModal.vue";

const selectedCampaign = ref(null);
const campaigns = ref([]);
const showModal = ref(false); // Controls the visibility of the dialog

const netSuiteApiCall = inject("netsuiteApi");

onMounted(async () => {
  try {
    const response = await netSuiteApiCall({
      task: "fetchCampaigns",
      campaign: "campaignId",
    });
    const data = await response.json();
    campaigns.value = data;
  } catch (error) {
    console.error("Error fetching campaigns:", error);
  }
});

const onRowSelect = (event) => {
  selectedCampaign.value = event.data;
  showModal.value = true;
};
</script>
<style lang="scss" scoped></style>
