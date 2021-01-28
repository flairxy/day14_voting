<template>
  <div>
    <h5>
      Create Ballot
      <span class="text-danger">Admin only (account[0])</span>
    </h5>

    <p>Current Ballot ID: {{ ballot ? ballot.id : "Unavailable" }}</p>
    <form>
      <div>
        <label class="mr-2" for="">Ballot Name: </label>
        <input
          type="text"
          placeholder="Enter Ballot name"
          v-model="ballotName"
        />
      </div>
      <div>
        <label class="mr-2" for="">Choices: </label>
        <input
          type="text"
          placeholder="choice1, choice2, choice3"
          v-model="choices"
        />
      </div>
      <div>
        <label class="mr-2" for="">Duration: </label>
        <input
          type="number"
          placeholder="duration in seconds"
          v-model="duration"
        />
      </div>
      <button class="btn btn-sm btn-primary" @click.prevent="createBallot">
        create ballot
      </button>
    </form>
    <hr />

    <h5>Vote</h5>
    <p>This operation can only be done by account1...account3</p>
    <form>
      <p>
        Current Ballot ID:
        <b>{{ currentBallotId >= 0 ? currentBallotId : "Does not exit" }}</b>
      </p>
      <p>
        Available Choices:
        <ul>
          <li v-for="choice in currentChoices" :key="choice.id">
             {{ choice.name }}: {{ choice.id }}
          </li>
        </ul>
      </p>
      <input type="number" v-model="currentChoice" placeholder="choice id" /><button
        @click.prevent="vote"
      >
        vote
      </button>
    </form>
    <hr>
    <h5>Get Results</h5>
    <p>This operation can only be done by the admin account[0]</p>
    <form>
        <p>
        <ul>
          <li v-for="result in results" :key="result.id">
             {{ result.name }}: {{ result.id }}
          </li>
        </ul>
      </p>
    <button class="btn btn-danger"
        @click.prevent="getResult"
      >
        Get Result
      </button>
    </form>
  </div>
</template>
<script>
import { mapGetters } from "vuex";
export default {
  data: () => ({
    ballot: undefined,
    ballotName: "",
    currentBallotId: undefined,
    currentBallot: undefined,
    duration: 120,
    choices: undefined,
    currentChoice: undefined,
    currentChoices: undefined,
    results: undefined,
  }),
  computed: {
    ...mapGetters("accounts", ["activeAccount"]),
    ...mapGetters("drizzle", ["drizzleInstance"]),
  },
  methods: {
    getResult() {
      this.drizzleInstance.contracts.Voting.methods
        .results(this.currentBallotId)
        .call()
        .then((res) => {
          this.results = res
          this.init();
        })
    },
    vote() {
      this.drizzleInstance.contracts.Voting.methods
        .vote(this.currentBallotId, this.currentChoice)
        .send()
        .then(() => {
          this.init();
        })
    },
    createBallot() {
      let choices = this.choices.split(",").map((choice) => {
        return choice.trim();
      });
      this.drizzleInstance.contracts.Voting.methods
        .createBallot(this.ballotName, choices, this.duration)
        .send()
        .then(() => {
          this.init();
        });
    },
    async init() {
      const contract = this.drizzleInstance.contracts.Voting;
      await contract.methods
        .nextBallotId()
        .call()
        .then((res) => {
          this.currentBallotId = res - 1;
        });
      if (this.currentBallotId >= 0) {
        contract.methods
          .getBallot(this.currentBallotId)
          .call()
          .then((res) => {
            this.currentChoices = res.choices;
          });
      }
    },
  },
  created() {
    this.init();
  },
};
</script>
