<template>
    <div class="m-md-auto">
        <b-modal id="modal-lg" size="lg" v-model="showModal" hide-footer hide-header centered>
            <div class="main-container">
                <div>
                    <button type="button" aria-label="Close" class="close" @click="hide">×</button>
                </div>
                <div>
                    <div class="profile-header">
                        <h3>ที่อยู่จัดส่ง</h3>
                        <h4>─────────────────────────────────────</h4>
                    </div>
                    <form @submit.prevent="submitForm">
                        <!-- container ชื่อ-นามสกุล -->
                        <div class="detail-container1">
                            <div class="detail-container1-name">
                                <label>ชื่อ <span class="text-danger">*</span></label>
                                <input type="text" placeholder="ชื่อ" v-model="form.firstName" />
                                <div class="text-danger" v-if="!$v.form.firstName.required">
                                    โปรดระบุชื่อ
                                </div>
                            </div>
                            <div class="detail-container1-last">
                                <label>นามสกุล <span class="text-danger">*</span></label>
                                <input type="text" placeholder="นามสกุล" v-model="form.lastName" />
                                <div class="text-danger" v-if="!$v.form.lastName.required">
                                    โปรดระบุนามสกุล
                                </div>
                            </div>
                        </div>

                        <!-- รายละเอียดที่อยู่ -->
                        <div class="detail-container2">
                            <label>รายละเอียดที่อยู่ <span class="text-danger">*</span></label>
                            <textarea
                                name=""
                                id=""
                                cols="85%"
                                rows="10"
                                placeholder="รายละเอียดที่อยู่"
                                v-model="form.street"
                            ></textarea>
                            <div class="text-danger" v-if="!$v.form.street.required">
                                โปรดระบุที่อยู่
                            </div>
                        </div>

                        <!-- จังหวัด-อำเภอ -->
                        <div class="detail-container3">
                            <div class="detail-container3-province">
                                <label>จังหวัด <span class="text-danger">*</span></label>
                                <select
                                    name="province"
                                    id="province"
                                    v-model="form.province"
                                    @change="getDistrict(form.province)"
                                >
                                    <option :value="null" disabled>กรุณาเลือกจังหวัด</option>
                                    <option
                                        v-for="(province, index) in provinces"
                                        :key="index"
                                        :value="province"
                                    >
                                        {{ province }}
                                    </option>
                                </select>
                            </div>
                            <div class="detail-container3-district">
                                <label>เขต/อำเภอ <span class="text-danger">*</span></label>
                                <select
                                    name="district"
                                    id="district"
                                    v-model="form.district"
                                    @change="getSubDistrict(form.province)"
                                    :disabled="!form.province"
                                >
                                    <option :value="null">กรุณาเลือกอำเภอ</option>
                                    <option
                                        v-for="(district, index) in districts"
                                        :key="index"
                                        :value="district"
                                    >
                                        {{ district }}
                                    </option>
                                </select>
                            </div>
                        </div>

                        <!-- ตำบล-รหัสไปรษณีย์ -->
                        <div class="detail-container4">
                            <div class="detail-container4-subdistrict">
                                <label>แขวง/ตำบล <span class="text-danger">*</span></label>
                                <select
                                    name="subdistrict"
                                    id="subdistrict"
                                    v-model="form.subDistrict"
                                    :disabled="!form.province || !form.district"
                                >
                                    <option
                                        v-for="(subDistrict, index) in subDistricts"
                                        :key="index"
                                        :value="subDistrict"
                                    >
                                        {{ subDistrict }}
                                    </option>
                                </select>
                            </div>
                            <div class="detail-container4-postnumber">
                                <label>รหัสไปรษณีย์ <span class="text-danger">*</span></label>
                                <input
                                    type="number"
                                    placeholder="รหัสไปรษณีย์"
                                    v-model="form.zipCode"
                                />
                                <div class="text-danger" v-if="!$v.form.zipCode.required">
                                    โปรดระบุรหัสไปรษณีย์
                                </div>
                            </div>
                        </div>

                        <!-- ปุ่ม -->
                        <div class="detail-container5">
                            <button>ย้อนกลับ</button>
                            <button @click="hide">บันทึก</button>
                        </div>
                    </form>
                </div>
            </div>
        </b-modal>
    </div>
</template>

<script>
import axios from "axios";
import { AddressApiService } from "@/services/address-api.service";
import { UserApiService } from "@/services/user-api.service";
import { required, minLength } from "vuelidate/lib/validators";
export default {
    name: "ModalAddress",
    props: ["getAddressList"],
    data: () => ({
        addressApiService: new AddressApiService(),
        userApiService: new UserApiService(),
        form: {
            firstName: "",
            lastName: "",
            street: "",
            province: "",
            district: "",
            subDistrict: "",
            zipCode: "",
        },
        showModal: false,
        provinces: [],
        districts: [],
        subDistricts: [],
    }),

    async created() {
        const defaultProvince = await this.getProvinces();
        if (defaultProvince === "") return;

        // const defaultDistrict = await this.getDistrict(defaultProvince);
        // console.log(defaultDistrict);

        // const defaultSubDistrict = await this.getSubDistrict(defaultProvince, defaultDistrict);
        // console.log(defaultSubDistrict);
    },

    methods: {
        async show(address) {
            console.log("address", address);

            if (!address) {
                this.resetForm();
            } else {
                this.form = address;
                if (this.form.id > 0) {
                    const defaultDistrict = await this.getDistrict();
                    console.log(defaultDistrict);

                    const defaultSubDistrict = await this.getSubDistrict();
                    console.log(defaultSubDistrict);
                }
            }

            this.showModal = true;
        },
        hide() {
            this.showModal = false;
        },

        // เลือกจังหวัด
        async getProvinces() {
            const res = await this.addressApiService.getProvinces();
            this.provinces = res.data.data.map((x) => x.province);
            if (this.provinces.length < 0) {
                return "";
            }
            return this.provinces[0];
        },

        // เลือกอำเภอ
        async getDistrict() {
            const res = await axios.get(
                `https://thaiaddressapi-thaikub.herokuapp.com/v1/thailand/provinces/${this.form.province}/district`
            );

            this.districts = res.data.data;
            if (this.districts.length < 0) {
                return "";
            }
            return this.districts[0];
        },

        //เลือกตำบล
        async getSubDistrict() {
            const res = await axios.get(
                `https://thaiaddressapi-thaikub.herokuapp.com/v1/thailand/provinces/${this.form.province}/district/${this.form.district}`
            );
            this.subDistricts = res.data.data;
        },
        async submitForm() {
            if (this.form.id > 0) {
                await this.userApiService.updateAddress(this.form);
            } else {
                await this.userApiService.createAddresses(this.form);
            }

            this.getAddressList();
            this.resetForm();

            this.$v.form.$touch();
            if (this.$v.form.$error) {
                return;
            }
        },

        // resetform หลังจาก submit
        resetForm() {
            this.form.firstName = "";
            this.form.lastName = "";
            this.form.street = "";
            this.form.province = "";
            this.form.district = "";
            this.form.subDistrict = "";
            this.form.zipCode = "";
        },
    },
    validations: {
        form: {
            firstName: {
                required,
            },
            lastName: {
                required,
            },
            street: {
                required,
            },
            zipCode: {
                required,
                minLength: minLength(5),
            },
        },
    },
};
</script>

<style scoped>
input {
    display: flex;
    width: 370px;
    height: 50px;
}
.profile-header {
    display: flex;
    align-items: center;
    padding-bottom: 10px;
    margin-top: 15px;
}

h4 {
    margin-left: 15px;
}

.detail-container1 {
    display: flex;
    justify-content: space-between;
    margin-top: 10px;
}

.detail-container2 {
    margin-top: 10px;
}

.detail-container1-name,
.detail-container1-last,
.detail-container3-province,
.detail-container3-district,
.detail-container4-subdistrict,
.detail-container4-postnumber {
    display: flex;
    flex-direction: column;
}

.detail-container3 {
    display: flex;
    margin-top: 10px;
    justify-content: space-between;
}

#province,
#district,
#subdistrict {
    width: 370px;
    height: 50px;
}
.detail-container4 {
    display: flex;
    margin-top: 10px;
    justify-content: space-between;
}

.detail-container5 {
    display: flex;
    justify-content: space-between;
    margin-top: 50px;
}
</style>
