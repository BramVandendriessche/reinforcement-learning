<template>
  <v-container fluid>
    <v-row justify="center" style="height: 3em;">
      <h1>{{message}}</h1>
    </v-row>
    <v-row style="padding-top: 5em">
      <v-col>
        <v-row
          v-for="row in rows"
          v-if="rows > 0"
          align="center"
          justify="center"
        >
          <v-card
            v-for="col in cols"
            v-if="cols > 0"
            class="ma-1 text-center"
            raised
            tile
            width="10em"
            height="10em"
            :color="getColor(state[row-1][col-1])"
          >
            <h1 v-if="state[row-1][col-1] !== 0" style="font-size: 5em; padding: 10%">{{state[row-1][col-1]}}</h1>
            <v-img :hidden="state[row-1][col-1] !== 0" :src="require('~/assets/sliding-puzzle/2tp.png')"></v-img>
          </v-card>
        </v-row>

      </v-col>
<!--      <v-col cols="12">-->
<!--        <v-row justify="center">-->
<!--          <v-col-->
<!--            cols="6"-->
<!--            md="2"-->
<!--          >-->
<!--            <v-btn block @click="shuffle" :disabled="shuffling">-->
<!--              Shuffle-->
<!--            </v-btn>-->
<!--          </v-col>-->

<!--          <v-col-->
<!--            cols="6"-->
<!--            md="2"-->
<!--          >-->
<!--            <v-btn block @click="solve" :disabled="shuffling">-->
<!--              Solve-->
<!--            </v-btn>-->
<!--          </v-col>-->
<!--        </v-row>-->
<!--      </v-col>-->
    </v-row>

<!--    <v-snackbar-->
<!--      top-->
<!--      v-model="snackbar"-->
<!--      :timeout="2000"-->
<!--    >-->
<!--      Solution not available for this grid. Reshuffling...-->
<!--    </v-snackbar>-->
  </v-container>
</template>

<script>
    export default {
        data () {
            return {
                // alignmentsAvailable: [
                //     'start',
                //     'center',
                //     'end',
                //     'baseline',
                //     'stretch',
                // ],
                // alignment: 'center',
                // dense: false,
                // justifyAvailable: [
                //     'start',
                //     'center',
                //     'end',
                //     'space-around',
                //     'space-between',
                // ],
                // justify: 'center',
                //
                // grid: [[1, 2], [3, 0]],
                //
                // solutions: [],
                //
                // snackbar: false,
                // count: 0,
                // solved: false,
                // shuffling: false,
                // indexShuffle: 1,

                buff: [],

                cols: 0,
                rows: 0,
                state: [],

                message: ''
            }
        },
        beforeMount() {
            if (this.$mqtt.connected) {
                this.$mqtt.subscribe('2tp/workshop/slidingpuzzle/state')
            }
            else this.message = 'Websocket not connected'
        },
        methods: {
            shuffle() {
                this.solved = false;
                this.indexShuffle = 0;
                this.shuffleLoop();
                this.shuffling = true
            },
            shuffleLoop() {
                setTimeout(() => {
                    const actions = ['Down', 'Up', 'Left', 'Right'];

                    let random = Math.floor(Math.random() * (+4 - +0)) + +0;
                    this.count +=1;

                    this.switchNumbers(actions[random], 'shuffle');
                    this.indexShuffle += 1;

                    if(this.indexShuffle < 60) {
                        this.shuffleLoop()
                    } else this.shuffling = false
                }, 200)

            },
            // async solve() {
            //
            //     this.solutions = await this.$axios.$get(`http://localhost:8881/solve?grid=[[${this.grid[0][0]}, ${this.grid[0][1]}], [${this.grid[1][0]}, ${this.grid[1][1]}]]`)
            //
            //     if (this.solutions.length !== 0) {
            //         this.count += 1
            //
            //         if (
            //             this.grid[0][0] === 1 &&
            //             this.grid[0][1] === 2 &&
            //             this.grid[1][0] === 3 &&
            //             this.grid[1][1] === 0
            //         ) {
            //             this.solved = true;
            //         } else {
            //             const solution = this.findAction();
            //
            //             if (solution === false) {
            //                 this.snackbar = true;
            //                 setTimeout(() => {
            //                     this.shuffle()
            //                 }, 1000)
            //             } else {
            //                 setTimeout(() => {
            //                     this.switchNumbers(solution.action, 'solve')
            //                 }, 1000)
            //             }
            //         }
            //     }
            // },
            findAction() {
                let solved = false;
                let solutionFound = null;
                for (let solution of this.solutions) {
                    if (
                        solution.configuration[0][0] === this.grid[0][0] &&
                        solution.configuration[0][1] === this.grid[0][1] &&
                        solution.configuration[1][0] === this.grid[1][0] &&
                        solution.configuration[1][1] === this.grid[1][1]
                    ) {
                        solved = true;
                        solutionFound = solution
                    }
                }

                if (solved) {
                    return solutionFound
                }
                else return false
            },
            getIndex(number) {
                if (this.grid[0][0] === number) {
                    return '0,0'
                    // return 1
                }
                else if (this.grid[0][1] === number) {
                    return '0,1'
                    // return 2
                }
                else if (this.grid[1][0] === number) {
                    return '1,0'
                    // return 3
                }
                else if (this.grid[1][1] === number) {
                    return '1,1'
                }
            },
            switchNumbers(action, method) {
                console.log(this.grid)
                const firstIndex = this.getIndex(0).split(',');
                let secondIndex = '';

                switch (action) {
                    case 'Up': secondIndex = '0,' + firstIndex[1];
                        break;
                    case 'Down': secondIndex = '1,' + firstIndex[1];
                        break;
                    case 'Left': secondIndex = firstIndex[0] + ',0';
                        break;
                    case 'Right': secondIndex = firstIndex[0] + ',1';
                        break;
                }

                console.log(secondIndex)
                if (secondIndex !== '') {
                    this.grid[parseInt(firstIndex[0])][parseInt(firstIndex[1])] = this.grid[parseInt(secondIndex.split(',')[0])][parseInt(secondIndex.split(',')[1])]
                    this.grid[parseInt(secondIndex.split(',')[0])][parseInt(secondIndex.split(',')[1])] = 0
                }

                if (method === 'solve') {
                    this.solve()
                }
            },
            getColor(value) {
                if (value === 0) {
                    return 'white'
                }
            },
            getMessage() {
                if (this.shuffling) {
                    return 'Shuffling...'
                }
                else if (this.solved) {
                    return 'Puzzle is solved!'
                }
                else return ''
            }
        },
        mqtt: {
            '2tp/workshop/slidingpuzzle/state' (data) {
                let json = JSON.parse(data)
                console.log(json)

                this.rows = json.nb_rows
                this.cols = json.nb_cols
                this.state = json.state
            }
        }
    }
</script>
