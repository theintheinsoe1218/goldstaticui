<template>
  <v-row no-gutters class="purchase-form-wrapper">
    <v-col cols="12" md="6" offset-md="3" class="form-column">
      <v-radio-group v-model="type" row class="radio-group-compact pa-2">
        <v-radio class="font-weight-bold" label="ဝယ်" value="PURCHASE" />
        <v-radio class="font-weight-bold" label="ရောင်း" value="SALE" />
        <v-radio class="font-weight-bold" label="ရွှေအပ်" value="GOLD" />
        <v-radio
          class="font-weight-bold"
          label="အပ်ရွှေ ငွေပြောင်း"
          value="EXCHANGE"
        />
        <v-radio class="font-weight-bold" label="ရွှေပြန်ယူ" value="RETURN" />
      </v-radio-group>

      <v-card outlined class="pa-3 main-form-card">
        <v-form v-model="valid">
          <v-row no-gutters>
            <v-col cols="12" md="6" class="pr-1 pb-1">
              <v-autocomplete
                v-model="purchaseDto.supplierDto"
                :items="userList"
                item-text="profileName"
                label="Customer Name"
                return-object
                dense
                outlined
                filled
                required
                hide-details
              >
                <template #append-outer>
                  <v-slide-x-reverse-transition mode="out-in">
                    <v-icon color="green" @click="clickUserDialog">
                      mdi-plus-circle
                    </v-icon>
                  </v-slide-x-reverse-transition>
                </template>
              </v-autocomplete>
            </v-col>

            <v-col cols="12" md="6" class="pl-1 pb-1">
              <v-menu v-model="receivedMenu" full-width max-width="290px">
                <template #activator="{ on }">
                  <v-text-field
                    v-model="purchaseDto.receivedDate"
                    label="Received Date"
                    v-on="on"
                    dense
                    outlined
                    filled
                    hide-details
                  />
                </template>
                <v-date-picker
                  v-model="receivedPicker"
                  color="primary"
                  :max="new Date().toISOString().substr(0, 10)"
                />
              </v-menu>
            </v-col>

            <v-col cols="12" class="mb-2 summary-display-row">
              <v-card flat class="summary-box">
                <v-row no-gutters>
                  <v-col cols="6" class="text-left py-1 px-2">
                    <strong
                      class="text-h6 font-weight-bold primary--text summary-text"
                    >
                      {{ purchaseDto.itemTransDto.qty }} ကျပ်
                      {{ purchaseDto.itemTransDto.qtype }} ပဲ
                      {{ purchaseDto.itemTransDto.qtyyway }} ရွေး
                    </strong>
                  </v-col>
                  <v-col cols="6" class="text-right py-1 px-2">
                    <span class="white--text balance-text">
                      အကြွေး( {{ goldBalance }} /
                      {{ moneyBalance | numberFormat }} )
                    </span>
                  </v-col>
                </v-row>
              </v-card>
            </v-col>

            <v-col cols="12" class="text-right pb-1">
              <v-radio-group
                v-model="unitType"
                row
                class="mb-0 float-right compact-unit-radio"
              >
                <v-radio label="ကျပ်" value="KYAT" dense hide-details />
                <v-radio label="Gram" value="GRAM" dense hide-details />
                <v-radio label="Kg" value="KG" dense hide-details />
              </v-radio-group>
            </v-col>

            <v-col cols="12" md="12">
              <v-row no-gutters class="weight-input-row">
                <template v-if="unitType === 'KYAT'">
                  <v-col cols="4" class="pr-1">
                    <v-text-field
                      v-model.number="qty"
                      label="ကျပ်"
                      type="number"
                      dense
                      outlined
                      filled
                      autofocus
                      hide-details
                    />
                  </v-col>
                  <v-col cols="4" class="px-1">
                    <v-text-field
                      v-model.number="qtype"
                      label="ပဲ"
                      type="number"
                      dense
                      outlined
                      filled
                      hide-details
                    />
                  </v-col>
                  <v-col cols="4" class="pl-1">
                    <v-text-field
                      v-model.number="qtyyway"
                      label="ရွေး"
                      type="number"
                      dense
                      outlined
                      filled
                      hide-details
                    />
                  </v-col>
                </template>

                <v-col cols="12" v-else-if="unitType === 'GRAM'">
                  <v-text-field
                    v-model.number="g"
                    label="Gram"
                    type="number"
                    dense
                    outlined
                    filled
                    autofocus
                    hide-details
                  />
                </v-col>

                <v-col cols="12" v-else-if="unitType === 'KG'">
                  <v-text-field
                    v-model.number="kg"
                    label="KG"
                    type="number"
                    dense
                    outlined
                    filled
                    autofocus
                    hide-details
                  />
                </v-col>
              </v-row>
            </v-col>

            <template
              v-if="
                type === 'SALE' || type === 'EXCHANGE' || type === 'PURCHASE'
              "
            >
              <v-col cols="12" md="6" class="pr-1 pb-1 pt-1">
                <v-text-field
                  v-model.number="purchaseDto.itemTransDto.price"
                  label="Unit Amount"
                  type="number"
                  dense
                  outlined
                  filled
                  min="0"
                  hide-details
                />
              </v-col>

              <v-col cols="12" md="6" class="pl-1 pb-1 pt-1">
                <v-text-field
                  v-model.number="purchaseDto.itemTransDto.amount"
                  label="Amount"
                  type="number"
                  dense
                  outlined
                  filled
                  min="0"
                  hide-details
                />
              </v-col>

              <v-col cols="12" md="6" class="pr-1 pb-1">
                <v-autocomplete
                  v-model="purchaseDto.transDto.bankTypeDto"
                  :items="bankList"
                  item-text="name"
                  label="Bank"
                  return-object
                  dense
                  outlined
                  filled
                  required
                  hide-details
                />
              </v-col>

              <v-col cols="12" md="6" class="pl-1 pb-1">
                <v-text-field
                  v-model.number="purchaseDto.transDto.payment"
                  label="Payment"
                  type="number"
                  dense
                  outlined
                  filled
                  hide-details
                />
              </v-col>
            </template>

            <v-col cols="12" class="pt-1 pb-2">
              <v-textarea
                v-model="purchaseDto.remark"
                label="Remark"
                dense
                outlined
                filled
                rows="1"
                hide-details
              />
            </v-col>

            <v-col cols="12" class="text-right">
              <v-btn
                color="blue"
                class="white--text"
                @click="savePurchase"
                :disabled="!isFormValid"
              >
                {{ saveOrupdate }}
              </v-btn>
            </v-col>
          </v-row>
        </v-form>
      </v-card>
    </v-col>

    <v-col>
      <v-bottom-sheet fullscreen scrollable v-model="uaDialog">
        <v-sheet class="information-window-v-sheet colorGrey">
          <userAccountForm
            :supplierType="supplierType"
            :user="user"
            :saveOrUpdate="saveOrupdate"
            @closeDialog="CloseDialog"
            @saveUserAccount="saveUa"
          />
        </v-sheet>
      </v-bottom-sheet>
    </v-col>
  </v-row>
</template>

<script>
import userService from "../service/UserAccountService";
import bankTypeService from "../service/BankTypeService";
import purchaseService from "../service/PurchaseService";
import userAccountForm from "../components/UserAccountForm.vue";

export default {
  components: { userAccountForm },
  data: () => ({
    supplierType: "su",
    saveOrupdate: "SAVE",
    selectedOne: {},
    bankDto: {},
    type: "SALE",
    valid: false,
    receivedMenu: false,
    receivedPicker: new Date().toISOString().substr(0, 10),
    userList: [],
    bankList: [],
    uaDialog: false,
    user: {},
    PE_IN_KYAT: 1 / 16,
    YWAY_IN_KYAT: 1 / 128,
    totalKyat: 0,
    kyatPayment: 0,
    unitType: "KYAT",
    qty: 0,
    qtype: 0,
    qtyyway: 0,
    g: 0,
    kg: 0,
    goldBalance: "0",
    moneyBalance: 0,
    purchaseDto: {
      supplierDto: {},
      itemTransDto: {
        qty: 0,
        qtype: 0,
        qtyyway: 0,
        price: 0,
        amount: 0,
        g: 0,
        kg: 0,
      },
      transDto: { bankTypeDto: {}, payment: 0 },
      receivedDate: "",
      remark: "",
    },
  }),

  mounted() {
    this.purchaseDto.receivedDate = this.formatDate(this.receivedPicker);
    this.userListMethod();
    this.bankListMethod();

    const purchaseId = this.$route.query.purchaseId;
    if (purchaseId) {
      this.purchaseId = purchaseId;
      this.getNewPurchaseId();
    }
  },

  methods: {
    formatDate(picker) {
      const [y, m, d] = picker.split("-");
      return `${d}-${m}-${y}`;
    },

    savePurchase() {
      this.purchaseDto.type = this.type;
      const action =
        this.saveOrupdate === "SAVE"
          ? purchaseService.addPurchase(this.purchaseDto)
          : purchaseService.updatePurchase(this.purchaseDto, this.purchaseId);

      action
        .then((response) => this.$router.push({ name: "npurchaselist" }))
        .catch((err) =>
          this.$swal("Fail!", err.response.data.message, "error")
        );
    },

    userListMethod() {
      userService
        .getUserList("CUSTOMER")
        .then((res) => {
          this.userList = res;
          this.purchaseDto.supplierDto = res[0];
        })
        .catch((err) =>
          this.$swal("Fail!", err.response.data.message, "error")
        );
    },

    bankListMethod() {
      bankTypeService
        .getBankType()
        .then((res) => {
          this.bankList = res;
          this.purchaseDto.transDto.bankTypeDto = res[0];
          this.saveOrupdate = "SAVE";
        })
        .catch((err) =>
          this.$swal("Fail!", err.response.data.message, "error")
        );
    },
    convertKyat() {
      const kyat = +this.qty || 0;
      const pe = +this.qtype || 0;
      const yway = +this.qtyyway || 0;
      const g = +this.g || 0;
      const kg = +this.kg || 0;
      let total = 0;

      if (this.unitType === "KYAT") {
        total = kyat + pe * this.PE_IN_KYAT + yway * this.YWAY_IN_KYAT;
      } else if (this.unitType === "GRAM") {
        this.gramToKyat(g);
        total = kyat + pe * this.PE_IN_KYAT + yway * this.YWAY_IN_KYAT;
      } else if (this.unitType === "KG") {
        this.kgToG(kg);
        total = kyat + pe * this.PE_IN_KYAT + yway * this.YWAY_IN_KYAT;
      }

      this.totalKyat = +total.toFixed(6);
      const { kyat: k, pe: p, yway: y } = this.kyatToKyatPeYway(this.totalKyat);
      Object.assign(this.purchaseDto.itemTransDto, {
        qty: k,
        qtype: p,
        qtyyway: y,
      });
      this.calculateKyatPayment();
    },

    calculateKyatPayment() {
      const price = +this.purchaseDto.itemTransDto.price || 0;
      const raw = (this.totalKyat || 0) * price;
      // console.log("raw " + raw);

      const rounded = Math.round(Math.round(raw) / 100) * 100;
      // console.log(rounded);

      this.kyatPayment = rounded;
      this.purchaseDto.itemTransDto.amount = rounded;
    },

    gramToKyat(gram) {
      const k = gram / 16.606;
      const res = this.kyatToKyatPeYway(k);
      Object.assign(this.purchaseDto.itemTransDto, res);
      Object.assign(this, res);
    },

    kyatToGram(kyat) {
      this.g = kyat * 16.3293;
      return this.g;
    },

    kgToG(kg) {
      this.gramToKyat(kg * 1000);
    },

    kyatToKyatPeYway(totalKyat) {
      const kyat = Math.floor(totalKyat);
      const peAll = (totalKyat - kyat) * 16;
      const pe = Math.floor(peAll);
      const yway = Math.round((peAll - pe) * 8);
      return { kyat, pe, yway };
    },

    getUserBalance() {
      userService
        .getAllBalance(this.purchaseDto.supplierDto.userAccountId)
        .then((res) => {
          this.moneyBalance = res.balance;
          const totalKyat =
            res.totalqty +
            res.totalqtype * this.PE_IN_KYAT +
            res.totalqtyyway * this.YWAY_IN_KYAT;
          const { kyat, pe, yway } = this.kyatToKyatPeYway(totalKyat);
          this.goldBalance = `${kyat} ကျပ် ${pe} ပဲ ${yway} ရွေး`;
        })
        .catch((err) =>
          this.$swal("Fail!", err.response.data.message, "error")
        );
    },

    getNewPurchaseId() {
      purchaseService
        .getNewPurchaseById(this.purchaseId)
        .then((res) => {
          this.purchaseDto = res;
          this.type = res.type;
          this.qty = res.itemTransDto.qty;
          this.qtype = res.itemTransDto.qtype;
          this.qtyyway = res.itemTransDto.qtyyway;
          this.saveOrupdate = "UPDATE";
        })
        .catch((err) =>
          this.$swal("Fail!", err.response.data.message, "error")
        );
    },

    clickUserDialog() {
      this.uaDialog = true;
      this.user = {};
    },
    CloseDialog() {
      this.uaDialog = false;
      this.user = {};
      this.saveOrupdate = "SAVE";
    },
    saveUa() {
      this.uaDialog = false;
      this.userListMethod();
    },
  },

  watch: {
    receivedPicker(val) {
      this.purchaseDto.receivedDate = this.formatDate(val);
    },
    qty: "convertKyat",
    qtype: "convertKyat",
    qtyyway: "convertKyat",
    g: "convertKyat",
    kg: "convertKyat",
    totalKyat: "calculateKyatPayment",
    "purchaseDto.itemTransDto.price": "calculateKyatPayment",
    "purchaseDto.itemTransDto.amount": "calculateKyatPayment",
    "purchaseDto.supplierDto": "getUserBalance",
    unitType: "convertKyat",
    type() {
      if (!this.purchaseId) {
        Object.assign(this.purchaseDto.itemTransDto, {
          qty: 0,
          qtype: 0,
          qtyyway: 0,
          price: 0,
          amount: 0,
        });
        this.purchaseDto.transDto.payment = 0;
        this.purchaseDto.remark = "";
      }
    },
    unitType(newVal, oldVal) {
      if (!oldVal || newVal === oldVal) return;

      // Reset Kyat/Gram/Kg inputs based on current totalKyat when switching
      const totalKyat =
        (this.purchaseDto.itemTransDto.qty || 0) +
        (this.purchaseDto.itemTransDto.qtype || 0) * this.PE_IN_KYAT +
        (this.purchaseDto.itemTransDto.qtyyway || 0) * this.YWAY_IN_KYAT;

      this.qty = 0;
      this.qtype = 0;
      this.qtyyway = 0;
      this.g = 0;
      this.kg = 0;

      if (newVal === "KYAT") {
        const { kyat, pe, yway } = this.kyatToKyatPeYway(totalKyat);
        this.qty = kyat;
        this.qtype = pe;
        this.qtyyway = yway;
        Object.assign(this.purchaseDto.itemTransDto, {
          qty: kyat,
          qtype: pe,
          qtyyway: yway,
        });
      } else if (newVal === "GRAM") {
        const g = totalKyat * 16.606;
        this.g = +g.toFixed(3);
        Object.assign(this.purchaseDto.itemTransDto, { g: this.g });
      } else if (newVal === "KG") {
        const g = totalKyat * 16.606;
        const kg = g / 1000;
        this.kg = +kg.toFixed(4);
        Object.assign(this.purchaseDto.itemTransDto, { kg: this.kg });
      }

      this.convertKyat();
    },
  },
  computed: {
    isFormValid() {
      const s = this.purchaseDto;
      const hasSupplier = s.supplierDto && s.supplierDto.userAccountId;
      const hasDate = !!s.receivedDate;
      const hasType = !!this.type;
      const hasWeight =
        (s.itemTransDto.qty || 0) > 0 ||
        (s.itemTransDto.qtype || 0) > 0 ||
        (s.itemTransDto.qtyyway || 0) > 0 ||
        (s.itemTransDto.g || 0) > 0 ||
        (s.itemTransDto.kg || 0) > 0;
      if (this.type === "SALE" || this.type === "EXCHANGE") {
        const hasPrice = (s.itemTransDto.price || 0) > 0;
        const hasAmount = (s.itemTransDto.amount || 0) > 0;
        const hasBank =
          s.transDto.bankTypeDto && s.transDto.bankTypeDto.bankTypeId;
        const hasPayment = (s.transDto.payment || 0) >= 0;
        return (
          hasSupplier &&
          hasDate &&
          hasType &&
          hasWeight &&
          hasPrice &&
          hasAmount &&
          hasBank &&
          hasPayment
        );
      }
      return hasSupplier && hasDate && hasType && hasWeight;
    },
  },
};
</script>

<style scoped>
/* Ensure the overall page/layout doesn't scroll and takes up full viewport height */
.purchase-form-wrapper {
  height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

/* Specific column for the form to ensure it centers and constraints height */
.form-column {
  flex-grow: 1; /* Allows the column to take available space */
  display: flex;
  flex-direction: column;
  padding-top: 10px; /* Small top padding */
  padding-bottom: 10px; /* Small bottom padding */
  max-height: 100vh;
}

/* Styling for the main card/form container */
.main-form-card {
  flex-grow: 1;
  overflow-y: hidden; /* Prevent internal scroll on the card itself */
  display: flex;
  flex-direction: column;
  background-color: #1e1e1e !important; /* Dark background */
  border: 1px solid #ffd700; /* Gold border */
}

/* Target the v-form inside the card to allow contents to fill space */
.main-form-card .v-form {
  overflow-y: auto; /* Allow scroll only if content truly overflows the constrained height */
  padding-right: 5px; /* Add slight padding for scrollbar visibility */
}

/* Compact spacing for the top radio group */
.radio-group-compact {
  padding: 0 8px;
  margin-top: 0;
  margin-bottom: 5px !important;
}

/* Reduce margin/padding for compact radio buttons */
.radio-group-compact >>> .v-radio {
  margin-right: 12px;
}

/* Summary Box Styling */
.summary-display-row {
  margin-bottom: 4px !important;
}
.summary-box {
  background-color: #000000 !important;
  border: 1px solid #ffd700 !important;
  border-radius: 4px;
}
.summary-text {
  font-size: 1.1rem !important; /* Slightly larger for main weight */
  color: #ffd700 !important; /* Gold color for main weight */
}
.balance-text {
  font-size: 0.9rem; /* Smaller for balance */
  color: white !important;
}

/* Compact unit radio buttons */
.compact-unit-radio {
  margin-bottom: 0 !important;
  height: 30px;
}
.compact-unit-radio >>> .v-radio {
  margin-top: 0 !important;
  padding-top: 0 !important;
}

/* Make text fields slightly more compact/less vertical space */
.v-input--dense .v-input__control {
  min-height: 36px !important;
}
.v-input--dense .v-text-field__details {
  margin-bottom: 0px !important;
}

/* Compact textarea */
.v-textarea textarea {
  line-height: 1.4rem;
}
</style>
