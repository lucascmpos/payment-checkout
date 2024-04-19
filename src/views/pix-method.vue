<template>
  <section>
    <h1>Pagamento via PIX</h1>
    <h2>Faça a leitura do QR CODE para finalizar o pagamento!</h2>
    <div>
      <form>
        <div>
          <label for="fullName">Nome completo</label>
          <input
            type="text"
            id="fullName"
            name="fullName"
            v-model="fullName"
            required
          />
        </div>
        <div>
          <label for="cpf">CPF do responsável</label>
          <input
            type="text"
            id="cpf"
            name="cpf"
            pattern="[0-9]{11}"
            v-model="cpf"
            required
          />
        </div>
      </form>
    </div>
    <div id="qrcode">
      <img v-if="qrCodeUrl" :src="qrCodeUrl" alt="QR Code" />
      <p class="timer" v-if="remainingTime > 0">
        Faltam {{ formatTime(remainingTime) }} para expirar
      </p>

      <Button
        :clickHandler="handleButtonClick"
        buttonText="Já fiz o pagamento"
      />
      <p class="error">{{ errorMessage }}</p>
    </div>
  </section>
</template>

<script>
import Button from "../components/button.vue";
import useQRCode from "../composables/useQRCode";

import { mapMutations } from "vuex";

export default {
  name: "PixMethod",
  components: {
    Button,
  },
  data() {
    return {
      qrCodeUrl: "",
      expirationTime: 10 * 60,
      remainingTime: 0,
      buttonClicked: false,
    };
  },
  computed: {
    fullName: {
      get() {
        return this.$store.state.fullName;
      },
      set(value) {
        this.updateFullName(value);
      },
    },
    cpf: {
      get() {
        return this.$store.state.cpf;
      },
      set(value) {
        this.updateCpf(value);
      },
    },
    errorMessage() {
      if (this.buttonClicked && (!this.fullName || !this.cpf)) {
        return "Por favor, preencha todos os campos.";
      } else {
        return "";
      }
    },
  },
  methods: {
    ...mapMutations(["updateFullName", "updateCpf"]),
    handleButtonClick() {
      this.buttonClicked = true;
      if (this.fullName && this.cpf) {
        this.$router.push({ name: "FinalStep" });
      }
    },
    clearErrorMessage() {
      const fullName = document.getElementById("fullName").value;
      const cpf = document.getElementById("cpf").value;

      if (fullName && cpf) {
        this.errorMessage = "";
      }
    },
    updateRemainingTime() {
      this.remainingTime = Math.max(
        0,
        this.expirationTime -
          Math.floor((Date.now() - this.qrCodeGeneratedTime) / 1000)
      );
    },
    formatTime(totalSeconds) {
      const minutes = Math.floor(totalSeconds / 60);
      const seconds = totalSeconds % 60;
      return `${minutes.toString().padStart(2, "0")}:${seconds
        .toString()
        .padStart(2, "0")}`;
    },
  },
  mounted() {
    const inputFields = document.querySelectorAll('input[type="text"]');
    inputFields.forEach((input) => {
      input.addEventListener("input", this.clearErrorMessage);
    });
    const qrCodeContent = "https://github.com/vitorlbarroso/teste-front-flame";
    const qrCodeOptions = {
      errorCorrectionLevel: "H",
      type: "image/jpeg",
      quality: 0.3,
      margin: 1,
      width: 200,
      color: {
        dark: "#000000",
        light: "#ffffff",
      },
    };

    const { generateQRCode } = useQRCode(qrCodeContent, qrCodeOptions);

    generateQRCode().then((url) => {
      this.qrCodeUrl = url;
      this.qrCodeGeneratedTime = Date.now();
      this.updateRemainingTime();
      this.timerInterval = setInterval(this.updateRemainingTime, 1000);
    });

    setInterval(async () => {
      try {
        this.qrCodeUrl = await generateQRCode();
        this.qrCodeGeneratedTime = Date.now();
        this.updateRemainingTime();
      } catch (error) {
        console.error("Error generating QR code:", error);
      }
    }, 600000);
  },
  beforeDestroy() {
    clearInterval(this.timerInterval);
  },
};
</script>

<style lang="scss" scoped>
@import "../components/styles/variables.scss";

section {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 465px;
  height: 416px;
  padding: 6%;
}

section > div {
  display: flex;
  flex-direction: column;
  align-content: center;
}
section > form {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  width: 100%;
}
form > div {
  display: flex;
  flex-direction: column;
  margin-bottom: 2px;
}

label {
  font-size: 12px;
  margin-top: 1%;
}

input {
  border-radius: 3px;
  border: 1px solid #757575;
  width: 100%;
  height: 24px;
}
h1 {
  @each $property, $value in $title-text {
    #{$property}: $value;
  }
}

h2 {
  width: 100%;
  font-size: 12px;
  font-weight: 600;
  color: #3a3a3a;
}
.timer {
  font-size: 12px;
  font-weight: 600;
  color: #09a42b;
}
#qrcode {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin-top: 5px;
}

#cpf {
  width: 50%;
}
.error {
  @each $property, $value in $error-text {
    #{$property}: $value;
  }
}
</style>