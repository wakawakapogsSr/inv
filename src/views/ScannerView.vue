<script setup>
	import { ref,computed } from 'vue'
	import { QrcodeStream } from 'vue-qrcode-reader'

	const showScanner = ref(false)

	const isValid = ref(undefined)
	const paused = ref(false)
	const result = ref([])

	const facingMode = ref('environment')
	const noRearCamera = ref(false)
	const noFrontCamera = ref(false)

	const validationPending = computed(() => isValid.value === undefined && paused.value)
	const validationSuccess = computed(() => isValid.value === true)
	const validationFailure = computed(() => isValid.value === false)


	const onError = console.error
	const resetValidationState = () => {
	  isValid.value = undefined
	}

	const onDetect = async ([firstDetectedCode]) => {
	  result.value.push(firstDetectedCode.rawValue)
	  paused.value = true

	  await timeout(1000)
	  isValid.value = true

	  // some more delay, so users have time to read the message
	  await timeout(2000)
	  paused.value = false

	  showScanner.value = false
	}

  const switchCamera = () => {
  	switch (facingMode.value) {
  		case 'environment':
  			facingMode.value = 'user'
  			break
  		case 'user':
  			facingMode.value = 'environment'
  			break
  	}
  }

	const timeout = (ms) => {
	  return new Promise((resolve) => {
	    window.setTimeout(resolve, ms)
	  })
	}

</script>

<template>
	<div>
    <button @click="showScanner = !showScanner">
      {{ showScanner ? 'Hide Scanner' : 'Show Scanner' }}
    </button>
    dd: {{ showScanner }} <br>
    <p class="error" v-if="noFrontCamera" > You don't seem to have a front camera on your device </p>
    <p class="error" v-if="noRearCamera" > You don't seem to have a rear camera on your device </p>

    {{ onError ? 'HasError' : 'NoError' }}
    <p v-if="!!!showScanner" class="decode-result">
      Last result: <b>{{ result }}</b>
    </p>
    <qrcode-stream
    	v-if="showScanner"
      :paused="paused"
      @detect="onDetect"
      @error="onError"
      @camera-on="resetValidationState"
    >
    	<button @click="switchCamera" class="in-button"> <img src="@/assets/logo.svg" alt="switch camera" /> </button>
      <div v-if="validationSuccess" class="validation-success" > This is a URL </div>
      <div v-if="validationFailure" class="validation-failure" > This is NOT a URL! </div>
      <div v-if="validationPending" class="validation-pending" > Long validation in progress... </div>
    </qrcode-stream>
  </div>
</template>

<style scoped>
.validation-success,
.validation-failure,
.validation-pending {
  position: absolute;
  width: 100%;
  height: 100%;

  background-color: rgba(255, 255, 255, 0.8);
  padding: 10px;
  text-align: center;
  font-weight: bold;
  font-size: 1.4rem;
  color: black;

  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}
.validation-success {
  color: green;
}
.validation-failure {
  color: red;
}

.in-button {
  position: absolute;
  left: 10px;
  top: 10px;
}
.in-button img {
  width: 50px;
  height: 50px;
}
.error {
  color: red;
  font-weight: bold;
}

</style>