<script setup>
import { FilterMatchMode } from 'primevue/api';
import { ref, onMounted, onBeforeMount, inject } from 'vue';
import { useToast } from 'primevue/usetoast';

const toast = useToast();

const netSuiteApiCall = inject("netsuiteApi");

const events = ref(null);
const eventDialog = ref(false);
const deleteEventDialog = ref(false);
const deleteEventsDialog = ref(false);
const event = ref({});
const selectedEvents = ref(null);
const dt = ref(null);
const filters = ref({});
const submitted = ref(false);
const statuses = ref([
    { label: 'Closed', value: 'closed' },
    { label: 'Planning', value: 'planning' },
    { label: 'Open', value: 'open' }
]);



onMounted(async () => {
    try {
        const response = await netSuiteApiCall({
            task: "fetchCampaigns",
            campaign: "campaignId",
        });
        const data = await response.json();
        events.value = data;
    } catch (error) {
        console.error("Error fetching campaigns:", error);
    }
});

const eventService = new eventService();

onBeforeMount(() => {
    initFilters();
});
onMounted(() => {
    eventService.getEvents().then((data) => (events.value = data));
});
const formatCurrency = (value) => {
    return value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
};



const openNew = () => {
    event.value = {};
    submitted.value = false;
    eventDialog.value = true;
};

const hideDialog = () => {
    eventDialog.value = false;
    submitted.value = false;
};

const saveEvent = () => {
    submitted.value = true;
    if (event.value.name && event.value.name.trim() && event.value.price) {
        if (event.value.id) {
            event.value.inventoryStatus = event.value.inventoryStatus.value ? event.value.inventoryStatus.value : event.value.inventoryStatus;
            events.value[findIndexById(event.value.id)] = event.value;
            toast.add({ severity: 'success', summary: 'Successful', detail: 'event Updated', life: 3000 });
        } else {
            event.value.id = createId();
            event.value.code = createId();
            event.value.image = 'event-placeholder.svg';
            event.value.inventoryStatus = event.value.inventoryStatus ? event.value.inventoryStatus.value : 'INSTOCK';
            event.value.push(event.value);
            toast.add({ severity: 'success', summary: 'Successful', detail: 'event Created', life: 3000 });
        }
        eventDialog.value = false;
        event.value = {};
    }
};

const editEvent = (editEvent) => {
    event.value = { ...editEvent };
    console.log(event);
    eventDialog.value = true;
};

const confirmDeleteEvent = (editEvent) => {
    event.value = editEvent;
    deleteEventDialog.value = true;
};

const deleteEvent = () => {
    event.value = event.value.filter((val) => val.id !== event.value.id);
    deleteEventDialog.value = false;
    event.value = {};
    toast.add({ severity: 'success', summary: 'Successful', detail: 'event Deleted', life: 3000 });
};

const findIndexById = (id) => {
    let index = -1;
    for (let i = 0; i < event.value.length; i++) {
        if (event.value[i].id === id) {
            index = i;
            break;
        }
    }
    return index;
};

const createId = () => {
    let id = '';
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
    for (let i = 0; i < 5; i++) {
        id += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return id;
};

const exportCSV = () => {
    dt.value.exportCSV();
};

const confirmDeleteSelected = () => {
    deleteEventsDialog.value = true;
};
const deleteSelectedEvents = () => {
    event.value = event.value.filter((val) => !selectedEvents.value.includes(val));
    deleteEventsDialog.value = false;
    selectedEvents.value = null;
    toast.add({ severity: 'success', summary: 'Successful', detail: 'event Deleted', life: 3000 });
};

const initFilters = () => {
    filters.value = {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS }
    };
};
</script>

<template>
    <div class="grid">
        <div class="col-12">
            <div class="card">
                <Toast />
                <Toolbar class="mb-4">
                    <template v-slot:start>
                        <div class="my-2">
                            <Button label="New" icon="pi pi-plus" class="p-button-success mr-2" @click="openNew" />
                            <Button label="Delete" icon="pi pi-trash" class="p-button-danger" @click="confirmDeleteSelected"
                                :disabled="!selectedEvents || !selectedEvents.length" />
                        </div>
                    </template>

                    <template v-slot:end>
                        <FileUpload mode="basic" accept="image/*" :maxFileSize="1000000" label="Import" chooseLabel="Import"
                            class="mr-2 inline-block" />
                        <Button label="Export" icon="pi pi-upload" class="p-button-help" @click="exportCSV($event)" />
                    </template>
                </Toolbar>

                <DataTable ref="dt" :value="event" v-model:selection="selectedEvents" dataKey="id" :paginator="true"
                    :rows="10" :filters="filters"
                    paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink CurrentPageReport RowsPerPageDropdown"
                    :rowsPerPageOptions="[5, 10, 25]"
                    currentPageReportTemplate="Showing {first} to {last} of {totalRecords} event" responsiveLayout="scroll">
                    <template #header>
                        <div class="flex flex-column md:flex-row md:justify-content-between md:align-items-center">
                            <h5 class="m-0">Manage event</h5>
                            <span class="block mt-2 md:mt-0 p-input-icon-left">
                                <i class="pi pi-search" />
                                <InputText v-model="filters['global'].value" placeholder="Search..." />
                            </span>
                        </div>
                    </template>

                    <Column selectionMode="multiple" headerStyle="width: 3rem"></Column>
                    <Column field="code" header="Code" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Code</span>
                            {{ slotProps.data.code }}
                        </template>
                    </Column>
                    <Column field="name" header="Name" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Name</span>
                            {{ slotProps.data.name }}
                        </template>
                    </Column>
                    <Column header="Image" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Image</span>
                            <img :src="'demo/images/event/' + slotProps.data.image" :alt="slotProps.data.image"
                                class="shadow-2" width="100" />
                        </template>
                    </Column>
                    <Column field="price" header="Price" :sortable="true" headerStyle="width:14%; min-width:8rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Price</span>
                            {{ formatCurrency(slotProps.data.price) }}
                        </template>
                    </Column>
                    <Column field="category" header="Category" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Category</span>
                            {{ slotProps.data.category }}
                        </template>
                    </Column>
                    <Column field="rating" header="Reviews" :sortable="true" headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Rating</span>
                            <Rating :modelValue="slotProps.data.rating" :readonly="true" :cancel="false" />
                        </template>
                    </Column>
                    <Column field="inventoryStatus" header="Status" :sortable="true"
                        headerStyle="width:14%; min-width:10rem;">
                        <template #body="slotProps">
                            <span class="p-column-title">Status</span>
                            <span
                                :class="'event-badge status-' + (slotProps.data.inventoryStatus ? slotProps.data.inventoryStatus.toLowerCase() : '')">{{
                                    slotProps.data.inventoryStatus }}</span>
                        </template>
                    </Column>
                    <Column headerStyle="min-width:10rem;">
                        <template #body="slotProps">
                            <Button icon="pi pi-pencil" class="p-button-rounded p-button-success mr-2"
                                @click="editEvent(slotProps.data)" />
                            <Button icon="pi pi-trash" class="p-button-rounded p-button-warning mt-2"
                                @click="confirmDeleteEvent(slotProps.data)" />
                        </template>
                    </Column>
                </DataTable>

                <Dialog v-model:visible="eventDialog" :style="{ width: '450px' }" header="event Details" :modal="true"
                    class="p-fluid">
                    <img :src="'demo/images/event/' + event.image" :alt="event.image" v-if="event.image" width="150"
                        class="mt-0 mx-auto mb-5 block shadow-2" />
                    <div class="field">
                        <label for="name">Name</label>
                        <InputText id="name" v-model.trim="event.name" required="true" autofocus
                            :class="{ 'p-invalid': submitted && !event.name }" />
                        <small class="p-invalid" v-if="submitted && !event.name">Name is required.</small>
                    </div>
                    <div class="field">
                        <label for="description">Description</label>
                        <Textarea id="description" v-model="event.description" required="true" rows="3" cols="20" />
                    </div>

                    <div class="field">
                        <label for="inventoryStatus" class="mb-3">Inventory Status</label>
                        <Dropdown id="inventoryStatus" v-model="event.inventoryStatus" :options="statuses"
                            optionLabel="label" placeholder="Select a Status">
                            <template #value="slotProps">
                                <div v-if="slotProps.value && slotProps.value.value">
                                    <span :class="'event-badge status-' + slotProps.value.value">{{ slotProps.value.label
                                    }}</span>
                                </div>
                                <div v-else-if="slotProps.value && !slotProps.value.value">
                                    <span :class="'event-badge status-' + slotProps.value.toLowerCase()">{{
                                        slotProps.value }}</span>
                                </div>
                                <span v-else>
                                    {{ slotProps.placeholder }}
                                </span>
                            </template>
                        </Dropdown>
                    </div>

                    <div class="field">
                        <label class="mb-3">Category</label>
                        <div class="formgrid grid">
                            <div class="field-radiobutton col-6">
                                <RadioButton id="category1" name="category" value="Accessories" v-model="event.category" />
                                <label for="category1">Accessories</label>
                            </div>
                            <div class="field-radiobutton col-6">
                                <RadioButton id="category2" name="category" value="Clothing" v-model="event.category" />
                                <label for="category2">Clothing</label>
                            </div>
                            <div class="field-radiobutton col-6">
                                <RadioButton id="category3" name="category" value="Electronics" v-model="event.category" />
                                <label for="category3">Electronics</label>
                            </div>
                            <div class="field-radiobutton col-6">
                                <RadioButton id="category4" name="category" value="Fitness" v-model="event.category" />
                                <label for="category4">Fitness</label>
                            </div>
                        </div>
                    </div>

                    <div class="formgrid grid">
                        <div class="field col">
                            <label for="price">Price</label>
                            <InputNumber id="price" v-model="event.price" mode="currency" currency="USD" locale="en-US"
                                :class="{ 'p-invalid': submitted && !event.price }" :required="true" />
                            <small class="p-invalid" v-if="submitted && !event.price">Price is required.</small>
                        </div>
                        <div class="field col">
                            <label for="quantity">Quantity</label>
                            <InputNumber id="quantity" v-model="event.quantity" integeronly />
                        </div>
                    </div>
                    <template #footer>
                        <Button label="Cancel" icon="pi pi-times" class="p-button-text" @click="hideDialog" />
                        <Button label="Save" icon="pi pi-check" class="p-button-text" @click="saveEvent" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteEventDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="event">Are you sure you want to delete <b>{{ event.name }}</b>?</span>
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteEventDialog = false" />
                        <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteEvent" />
                    </template>
                </Dialog>

                <Dialog v-model:visible="deleteEventsDialog" :style="{ width: '450px' }" header="Confirm" :modal="true">
                    <div class="flex align-items-center justify-content-center">
                        <i class="pi pi-exclamation-triangle mr-3" style="font-size: 2rem" />
                        <span v-if="event">Are you sure you want to delete the selected event?</span>
                    </div>
                    <template #footer>
                        <Button label="No" icon="pi pi-times" class="p-button-text" @click="deleteEventsDialog = false" />
                        <Button label="Yes" icon="pi pi-check" class="p-button-text" @click="deleteSelectedEvents" />
                    </template>
                </Dialog>
            </div>
        </div>
    </div>
</template>

<style scoped lang="scss"></style>
