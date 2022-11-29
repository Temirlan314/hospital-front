<script>
import DatePicker from "vue2-datepicker";
import Switches from "vue-switches";
import Multiselect from "vue-multiselect";
import NumberInputSpinner from "vue-number-input-spinner";
/**
 * Starter component
 */
export default {
  head() {
    return {
      title: `Hospital`,
    };
  },
  components: {
    DatePicker,
    Switches,
    Multiselect,
    NumberInputSpinner,
  },
  async created() {
    let specializations = await this.getSpecializations();
    this.specializations = specializations.data;
    let doctors = await this.getAllDoctors();
    this.doctors = doctors.data;

    for (const index in this.specializations) {
      let doctor = await this.getDoctors(this.specializations[index].id);
      this.specializations[index].doctors = doctor.data;
    }

    console.log(specializations, doctors, "RESPONSE");
  },
  methods: {
    async getSpecializations() {
      return this.$axios.get(`http://127.0.0.1:8000/specializations/`, {
        headers: {
          "X-Requested-With": "XMLHttpRequest",
          "Access-Control-Allow-Origin": "*",
        },
      });
    },
    async getAllDoctors() {
      return this.$axios.get(`http://127.0.0.1:8000/doctors/`, {
        headers: {
          "X-Requested-With": "XMLHttpRequest",
          "Access-Control-Allow-Origin": "*",
        },
      });
    },
    async getDoctors(id) {
      return this.$axios.get(
        `http://127.0.0.1:8000/specializations/${id}/doctors/`,
        {
          headers: {
            "X-Requested-With": "XMLHttpRequest",
            "Access-Control-Allow-Origin": "*",
          },
        }
      );
    },
    async getDoctorTimeSlotsFromSelect() {
      if (this.selectedDoctor) {
        let doctor = await this.getDoctorTimeSlots(this.selectedDoctor.id);
        this.doctor = doctor.data;
        console.log(doctor);
      }
    },
    async getDoctorTimeSlots(id) {
      return this.$axios.get(`http://127.0.0.1:8000/time-slots/${id}/`, {
        headers: {
          "X-Requested-With": "XMLHttpRequest",
          "Access-Control-Allow-Origin": "*",
        },
      });
    },
    customLabel(item) {
      return `${item.name} ${item.middlename} ${item.surname}`;
    },
    async openTab(newTabIndex, prevTabIndex) {
      if (this.specializations && this.specializations[newTabIndex]) {
        let selectedTab = await this.getDoctors(
          this.specializations[newTabIndex].id
        );
        this.selectedTab = selectedTab.data
      }
    },
  },
  data() {
    return {
      title: "Appointments",
      specializations: null,
      doctors: null,
      doctor: null,
      selectedDoctor: null,
      selectedTab: null,
      items: [
        {
          text: "Appointments",
          href: "/",
          active: true,
        },
      ],
    };
  },
  middleware: "router-auth",
};
</script>

<template>
  <div>
    <PageHeader :title="title" :items="items" />
    <div class="card p-3">
      <div class="row" v-if="specializations">
        <div class="col-12">
          <p class="h3">Search by name:</p>
        </div>
        <div class="col-12">
          <div class="row p-2">
            <div class="col-12 mb-4" v-if="doctors">
              <multiselect
                v-model="selectedDoctor"
                :options="doctors"
                :customLabel="customLabel"
                trackBy="name"
                placeholder="Search..."
                select-label="Select"
                deselect-label="Deselect"
                selected-label="Selected"
                :preserveSearch="true"
                :clearOnSelect="false"
                @input="getDoctorTimeSlotsFromSelect()"
              ></multiselect>
            </div>
            <div class="col-12" v-if="doctor && doctor.length">
              <p class="font-weight-bold">Busy time-slots:</p>
              <b-table
                hover
                striped
                table-class="table table-centered w-100 table-hover"
                :items="doctor"
              ></b-table>
            </div>
            <div class="col-12 font-weight-bold" v-else-if="doctor">
              No busy time-slots
            </div>
            <div class="col-12 font-weight-bold" v-else>
              No doctor is selected
            </div>
          </div>
        </div>
        <div class="col-12 mt-4">
          <div class="row p-2">
            <div class="col-12">
              <p class="h3">Search by specializations:</p>
            </div>
            <b-tabs
              nav-class="nav-tabs nav-bordered"
              class="col-12"
              @activate-tab="openTab"
            >
              <b-tab
                :title="item.specialization"
                v-for="(item, index) in specializations"
                :key="index"
              >
                <b-table
                  v-if="selectedTab"
                  hover
                  striped
                  table-class="table table-centered w-100 table-hover"
                  :items="selectedTab"
                ></b-table>
              </b-tab>
            </b-tabs>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
