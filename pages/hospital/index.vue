<script>
import DatePicker from "vue2-datepicker";
import Switches from "vue-switches";
import Multiselect from "vue-multiselect";
import NumberInputSpinner from "vue-number-input-spinner";
import moment from "moment";

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
  computed: {
    timeSlots() {
      let selectedDate = this.formatDate(this.selectedDate);
      if (this.doctor && this.doctor && this.selectedDate) {
        console.log(selectedDate);
        let busySlots = [];
        for (const index in this.doctor) {
          console.log(this.doctor[index].date, selectedDate);

          if (this.doctor[index].date === selectedDate) {
            busySlots.push(this.doctor[index].timeslot);
          }
        }
        let oldList = [
          "09:00 – 09:30",
          "09:30 – 10:00",
          "10:00 – 10:30",
          "10:30 – 11:00",
          "11:00 – 11:30",
          "11:30 – 12:00",
          "12:00 – 12:30",
          "12:30 – 13:00",
          "14:00 – 14:30",
          "14:30 – 15:00",
          "15:00 – 15:30",
          "15:30 – 16:00",
          "16:00 – 16:30",
          "16:30 – 17:00",
          "17:00 – 17:30",
          "17:30 – 18:00",
        ];
        let list = [];
        for (const item of oldList) {
          let check = false;
          for (const slot of busySlots) {
            console.log(slot, item, slot === item);
            if (slot === item) {
              check = true;
              break;
            }
          }
          if (!check) {
            list.push(item);
          }
        }
        return list;
      }
      return [];
    },
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
      console.log("ASKSK");
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
    async makeAppointment(payload) {
      return this.$axios.post(`http://127.0.0.1:8000/requests/`, payload, {
        headers: {
          "X-Requested-With": "XMLHttpRequest",
          "Access-Control-Allow-Origin": "*",
        },
      });
    },
    async handleSubmit() {
      if (
        this.selectedDate &&
        this.selectedTimeSlot &&
        this.phone &&
        this.surname &&
        this.surname.length &&
        this.name &&
        this.name.length
      ) {
        let payload = {
          timeslot: {
            date: this.formatDate(this.selectedDate),
            timeslot: this.selectedTimeSlot,
            doctor: this.selectedDoctor.id,
          },
          name: this.name,
          surname: this.surname,
          contact_number: this.phone,
        };
        try {
          await this.makeAppointment(payload);
          this.succes = true;
        } catch (e) {
          console.log(e);
          this.error.push(1);
        }
      }
    },
    formatDate(date) {
      var moment = require("moment");

      if (!date) {
        return "No date";
      }
      let momentDate = moment(date);
      let result = momentDate.format("YYYY-MM-DD");
      return result;
    },
    customLabel(item) {
      return `${item.name} ${item.middlename} ${item.surname}`;
    },
    timeSlotFilter(item) {
      if (this.doctor && this.doctor && this.selectedDate) {
        let busySlots = [];
        for (const index in this.doctor) {
          if (this.doctor[index].date == this.selectedDate) {
            busySlots.push(this.doctor[index].timeslot);
          }
        }

        for (const slot of busySlots) {
          if (slot == item) {
            return item + " not available";
          }
        }
      }
      return item + " available";
    },
    async openTab(newTabIndex, prevTabIndex) {
      if (this.specializations && this.specializations[newTabIndex]) {
        let selectedTab = await this.getDoctors(
          this.specializations[newTabIndex].id
        );
        this.selectedTab = selectedTab.data;
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
      selectedDate: null,
      selectedTimeSlot: null,
      error: [],
      success: false,
      phone: "",
      name: "",
      surname: "",
      items: [
        {
          text: "Appointments",
          href: "/",
          active: true,
        },
      ],

      selectedTabFields: [
        {
          key: "photo",
          label: "Photo",
        },
        {
          key: "name",
          label: "Name",
          sortable: true,
        },
        {
          key: "experience",
          label: "Experience",
          sortable: true,
        },
        {
          key: "contact_number",
          label: "Contact Number",
        },
      ],
      momentFormat: {
        stringify: (date) => {
          var moment = require("moment");
          return date ? moment(date).format("YY-MM-DD") : "";
        },
      },
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
            <div class="col-12" v-if="doctor">
              <div class="row">
                <div class="col-4 mb-3">
                  <p>Appointment date:</p>
                  <DatePicker
                    v-model="selectedDate"
                    :formatter="momentFormat"
                  />
                </div>
                <div class="col-4 mb-3">
                  <p>Time:</p>

                  <multiselect
                    v-model="selectedTimeSlot"
                    :options="timeSlots"
                    trackBy="name"
                    placeholder="Time slots"
                    select-label="Select"
                    deselect-label="Deselect"
                    selected-label="Selected"
                    :preserveSearch="true"
                    :clearOnSelect="false"
                    :searchable="false"
                  ></multiselect>
                </div>
                <div class="col-4"></div>
                <div class="col-2">
                  <p>Name</p>
                  <input
                    type="text"
                    class="form-control"
                    placeholder="Name"
                    v-model="name"
                  />
                </div>
                <div class="col-2">
                  <p>Surname</p>
                  <input
                    type="text"
                    class="form-control"
                    placeholder="Surname"
                    v-model="surname"
                  />
                </div>
                <div class="col-4">
                  <p>Phone number</p>
                  <input
                    v-model="phone"
                    type="text"
                    class="form-control"
                    v-mask="'+7 (###) ###-####'"
                    placeholder="+7 (###) ###-####"
                  />
                </div>
                <div
                  class="col-2 btn btn-primary mb-3 mt-4"
                  @click="handleSubmit"
                  style="height: 38px"
                >
                  Make an appointment
                </div>
                <div class="col-2" v-if="error && error.length">
                  <b-alert show dismissible variant="danger" v-for="er in error"
                    >Something went wrong!</b-alert
                  >
                </div>
                <div v-if="success">
                  <b-alert
                    show
                    dismissible
                    variant="success"
                    style="position:sticky;"
                    >Your appointment has been sent</b-alert
                  >
                </div>
                <Portlet class="col-10 row" headertitle="Busy time-slots list:" @load="getDoctorTimeSlotsFromSelect">
                  <div class="col-12" v-if="doctor && doctor.length">
                    <p class="font-weight-bold"></p>
                    <b-table
                      hover
                      striped
                      table-class="table table-centered w-100 table-hover"
                      :items="doctor"
                    ></b-table>
                  </div>

                  <div class="col-12 font-weight-bold mb-4 ml-2" v-else-if="doctor">
                    No busy time-slots
                  </div>
                  <div class="col-12 font-weight-bold mb-4 ml-2" v-else>
                    No doctor is selected
                  </div></Portlet
                >
              </div>
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
                  v-if="selectedTab && selectedTab.length"
                  hover
                  striped
                  table-class="table table-centered w-100 table-hover"
                  :items="selectedTab"
                  :fields="selectedTabFields"
                >
                  <template v-slot:cell(photo)="data">
                    <div class="media">
                      <img
                        v-if="data.item.photo"
                        :src="data.item.photo"
                        alt="table-user"
                        class="mr-3 rounded-circle avatar-sm"
                      />
                      <div v-if="!data.item.photo" class="avatar-sm mr-3">
                        <div
                          class="avatar-title rounded-circle bg-soft-primary font-weight-medium text-primary"
                        >
                          {{ data.item.name.charAt(0) }}
                        </div>
                      </div>
                    </div>
                  </template>

                  <template v-slot:cell(name)="data">
                    {{ data.item.name + " " }}{{ data.item.middlename + " "
                    }}{{ data.item.surname }}
                  </template>
                  <template v-slot:cell(experience)="data">
                    {{ data.item.experience + " "
                    }}{{ data.item.experience > 1 ? "years" : "year" }}
                  </template>
                </b-table>
                <div v-else>
                  No doctors
                </div>
              </b-tab>
            </b-tabs>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
