<template>
  <div id="app">
    <div class="header"></div>
    <div class="content">
      <h1>WABC Membership</h1>
      <h2>Member Benefits</h2>
      <ul>
        <li>Warrior Level - $250: Receive Family Sports Pass** good for one school year for all home games for those family members living in the same household (excludes IHSAA tournament games or special invitational games), one insulated cooler*, one Warrior Pride t-shirt and one decal</li>
        <li>Maroon Level - $150: Receive Individual Sports Pass** good for one school year for all home games for one individual (excludes IHSAA tournament games or special invitational games), one camp chair*, one Warrior Pride t-shirt and one decal</li>
        <li>Green Level - $50: Receive 2 single game sports passes**, one Warrior Pride t-shirt and one decal</li>
      </ul>
      <p><i>* Items exclusively available to WABC Members.<br>** All sports passes are good for one year from time of purchase and for home games only - not any type of tournament. Family Sports Passes cover those living in the same household.</i></p>
      <h2>Team Benefits</h2>
      <p>ALL Teams Benefit from Fundraising and Fees<br>Money Available for Expenses and Needs<br>Coaches and Teams Can Determine How to Spend Money</p>
      <form v-on:submit.prevent>
        <h2>About You</h2>
        <label for="firstname">First Name:</label>
        <input type="text" id="firstname" v-model="firstname" required>
        <label for="lastname">Last Name:</label>
        <input type="text" id="lastname" v-model="lastname" required>
        <label for="address">Address:</label>
        <input type="text" id="address" v-model="address" required>
        <label for="city">City:</label>
        <input type="text" id="city" v-model="city" required>
        <label for="state">State:</label>
        <input type="text" id="state" v-model="state" required>
        <label for="zipcode">Zipcode:</label>
        <input type="text" id="zipcode" v-model="zipcode" required>
        <label for="email">Email:</label>
        <input type="email" id="email" v-model="email" required>
        <label for="phone">Phone:</label>
        <input type="tel" id="phone" v-model="phone" required>
        <h2>About Your Membership</h2>
        <h3>Choose your membership level:</h3>
        <input type="radio" id="warrior" v-model="level" value="warrior" required>
        <label for="warrior">Warrior</label>
        <input type="radio" id="maroon" v-model="level" value="maroon" required>
        <label for="maroon">Maroon</label>
        <input type="radio" id="green" v-model="level" value="green" required>
        <label for="green">Green</label>
        <label for="students">Your Students:<br>If applicable, please list your student's name, year of graduation and any sports they played.</label>
        <textarea id="students" v-model="students" rows="5"></textarea>
        <h2>Become a Booster Club Volunteer!</h2>
        <p>The Warrior Athletic Booster Club is always looking for volunteers to work on committees and provide support for our fundraising. Please select your volunteer preference below:
YES! I would like to be involved in the WABC committees and activities. Please contact me for more information.

Maybe. I would like more information on current needs of a volunteer.

I cannot commit to volunteer activities at this time, but have ideas for fundraising projects/activities that we might want to consider for the future.
        </p>
        <h2>Payment</h2>
        <label for="card">Credit Card #:</label>
        <input type="text" id="card" v-model="card" required>
        <label for="expiration">Expiration:</label>
        <input type="text" id="expiration" v-model="expiration" required>
        <label for="code">Security Code:</label>
        <input type="text" id="code" v-model="code" required>
        <label for="amount">Amount: $</label>
        <input type="number" id="amount" v-model="amount" min="20" value="20" required>
        <div class="center">
          <div class="message">{{message}}</div>
          <div v-show="message === 'Processing...'">
            <img src="@/assets/ring-spinner.gif" alt="Spinner">
          </div>
          <div v-if="showSubmit">
            <input type="submit" :disabled="invalid" @click="register"><br>
            <span v-if="invalid">Form is incomplete or invalid.</span><br><br>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'app',
  data: () => {
    return {
      lastname: '',
      firstname: '',
      address: '',
      city: '',
      state: '',
      zipcode: '',
      email: '',
      phone: '',
      level: '',
      students: '',
      card: '',
      expiration: '',
      code: '',
      amount: '250',
      message: '',
      showSubmit: true
    }
  },
  computed: {
    invalid() {
      let invalid = false
      if (this.lastname === ''
        || this.firstname === ''
        || this.address === ''
        || this.city === ''
        || this.state === ''
        || this.zipcode === ''
        || this.email === ''
        || this.phone === ''
        || this.level === ''
        || this.students === ''
        || this.card === ''
        || this.expiration === ''
        || this.code === ''
        || this.amount === '') invalid = true
      if (parseFloat(this.amount) < 50.0) invalid = true
      return invalid
    }
  },
  methods: {
    register() {
      this.showSubmit = false
      this.message = 'Processing...'
      let html = '<html><body>'
      html += '<h1>WABC Membership</h1>'
      html += '<h2>Your Membership Details</h2>'
      // html += `<ul><li>Year: ${this.year}</li><li>Make: ${this.make}</li><li>Model: ${this.model}</li></ul>`
      // html += '<h2>Thank you so much for signing up to be a part of this special event honoring our volunteers!</h2><p>Please check in between 5-5:45pm at Covenant Christian High School. We look forward to seeing you on July 19!</p>'
      html += '</body></html>'
      let data = {
        firstName: this.firstname,
        lastName: this.lastname,
        address: this.address,
        city: this.city,
        state: this.state,
        zip: this.zipcode,
        email: this.email,
        phone: this.phone,
        level: this.level,
        students: this.students,
        ccNumber: this.card,
        ccExpiration: this.expiration,
        ccCode: this.code,
        amount: this.amount,
        description: 'WABC Membership',
        html
      }
      axios
        .post('https://us-central1-my-covenant.cloudfunctions.net/creditCard', {
          data
        })
        .then(result => {
          this.message = result.data.description
          if (result.data.responseCode != 1) {
            this.message += ' Please fix your information above and try again.'
            this.showSubmit = true
          }
          if (result.data.transactionId) {
            data.transactionId = result.data.transactionId
            axios
              .post('https://us-central1-my-covenant.cloudfunctions.net/wabc', {
                data
              })
              .then(result => {
                this.message = result.data
              })
              .catch(() => {
                this.message = 'Could not complete your membership.'
              })
          }
        })
        .catch(() => {
          this.message = 'This operation is currently unavailable.'
        })
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Oswald');
@import url('https://fonts.googleapis.com/css?family=Work+Sans');

html, body {
  background: #080910;
  color: #fff;
  font-family: 'Work Sans', sans-serif;
  margin: 0;
}

h1 {
  color: #4a6930;
  font-family: Oswald, sans-serif;
  text-transform: uppercase;
}

input {
  display: inline-block;
  font-family: 'Work Sans', sans-serif;
  font-size: 1em;
  padding: 0.5vw;
  width: 50vw;
}
input[type=submit] {
  background: #eee;
  border: solid 1px #777;
  border-radius: 1vw;
  display: inline-block;
  font-size: 1.5em;
  margin-top: 5vh;
  padding: 1vw;
  text-transform: uppercase;
  width: auto;
}
input[type=submit]:disabled {
  border: solid 1px #ccc;
}
label {
  display: inline-block;
  margin: 1vw;
  text-align: right;
  width: 30vw;
}
span {
  color: red;
  font-style: italic;
}
textarea {
  font-family: 'Work Sans', sans-serif;
  font-size: 1em;
  margin-top: 2px;
  padding: 0.5vw;
  vertical-align: middle;
  width: 50vw;
}

.content {
  margin: 2vw;
}

.header {
  background-image: url('./assets/WABC.jpg');
  background-position: center;
  background-size: contain;
  height: 30vw;
  background-repeat: no-repeat;
}
</style>
