<template>

<v-container fluid>

<v-layout row wrap>


    <v-flex xs12 md3>

        <v-card >

            <v-card-title class="primary">
                <div>
                    <h1 class="title white--text">Payment Plan</h1>
                </div>
            </v-card-title>

            <v-card-text>

                <v-form>

                    <span class="subheading">Payment Plan Type</span>

                    <v-tooltip right max-width="250">
                        <v-btn slot="activator" color="primary" flat icon><v-icon>fa-question-circle</v-icon></v-btn>
                        <span>The order in which loans are paid off, either focusing on largest interest rate, or lowest principal. The <b>Avalanche</b> method results in the least amount of interest paid. The <b>Snowball</b> method results in paying off smaller loans faster.</span>
                    </v-tooltip>

                    <v-checkbox v-model="paymentMethod" value="avalanche" label="Highest Intrest Rate (Avalance)" />
                    <v-checkbox v-model="paymentMethod" value="snowball" label="Lowest Principal (Snowball)" />

                </v-form>

                <v-divider /> 

                <v-form>

                    <span class="subheading">Monthly Payment</span>

                    <v-tooltip right max-width="250">
                        <v-btn slot="activator" color="primary" flat icon><v-icon>fa-question-circle</v-icon></v-btn>
                        <span>The amount dedicated to paying off your loans. Must be greater than all loans' minimum payments.</span>
                    </v-tooltip>

                    <v-text-field placeholder="$50" solo  />

                </v-form>


            </v-card-text>

        </v-card>

    </v-flex>

    <v-flex v-show="loans.length == 0" xs12 md9 px-2>

        <v-jumbotron height="100%" color="primary">
            <v-container fill-height>
                <v-layout align-center>
                    <v-flex>

                        <h3 class="display-2 white--text">IN DEBT? NEED A LIFE-RAFT?</h3>

                        <v-divider class="my-3 white white--text"></v-divider>

                        <span class="subheading white--text">raft is a loan calculator that helps you pay off your debts.<br /><br />Discover an optimized payment plan to minimize interest and get debt free as fast as possible.</span>

                        <br />
                        <br />

                        
                        <p class="text-xs-center">
                        <v-dialog v-model="showAddLoan" width="500">
                            <v-btn slot="activator">Enter a debt</v-btn>

                            <v-card>
                                <v-card-title class="primary">
                                    <div>
                                        <h1 class="title white--text">Add a Debt</h1>
                                    </div>
                                </v-card-title>

                                <v-card-text>

                                    <v-form>

                                        <v-text-field @input="toggleInputError" v-model="input.name" label="Debt Name" placeholder="Federal Student Loan, Chase Credit Card" />
                                        <v-text-field @input="toggleInputError" v-model="input.principal" label="Principal Remaining ($)" placeholder="$120,000" />
                                        <v-text-field @input="toggleInputError" v-model="input.interest_rate" label="Interest Rate (%)" placeholder="28.9%" />
                                        <v-text-field @input="toggleInputError" v-model="input.monthtly_minimum" label="Monthly Minimum Payment ($)" placeholder="$1000.00" />
                                        <v-text-field @input="toggleInputError" v-model="input.debt_color" label="Color" placeholder="red" />

                                        <v-slide-y-transition> 
                                            <p v-show="showInputError" class="subheading red--text">Invalid Values!</p>
                                        </v-slide-y-transition>

                                    </v-form>

                                </v-card-text>

                                <v-card-actions>
                                    <v-layout>
                                        <v-flex />
                                        <v-flex shrink>
                                            <v-btn @click="cancelAddLoan">Cancel</v-btn>
                                            <v-btn @click="addLoan" color="primary">Save</v-btn>
                                        </v-flex>
                                    </v-layout>
                                </v-card-actions>

                            </v-card>

                        </v-dialog>
                        </p>

                    </v-flex>
                </v-layout>
            </v-container>
        </v-jumbotron>

    </v-flex>

    <v-flex v-show="loans.length > 0" text-xs-center xs12 md9 px-4>

        <line-chart :chartData="chartData" :options="chartOptions"></line-chart>

    </v-flex>


</v-layout>
    
<v-divider class="my-5" />

<v-layout row wrap>


    <v-flex>

        <v-card >

            <v-card-title class="primary">
                <v-layout row wrap style="width:100%;">
                    <v-spacer />
                    <v-flex>
                        <h1 class="title white--text">My Debts</h1>
                    </v-flex>
                    <v-flex shrink>
                        <v-btn @click="showAddLoan = true" class="ma-0" color="white" icon flat><v-icon>add</v-icon></v-btn>
                    </v-flex>
                </v-layout>
            </v-card-title>

            <v-card-text>

                <h1 v-show="loans.length == 0" class="subheading text-xs-center">No Debts? Add some here to get advice on how to pay them off!</h1>

                <v-data-table
                    v-show="loans.length > 0"
                    :headers="headers"
                    :items="loans"
                    class="elevation-2">
                    <template slot="items" slot-scope="props">
                        <td>{{ props.item.name }}</td>
                        <td class="text-xs-center">{{ props.item.principal | displayMoney }}</td>
                        <td class="text-xs-center">{{ props.item.interest_rate }}%</td>
                        <td class="text-xs-center">{{ props.item.monthtly_minimum | displayMoney }}</td>
                    </template>
                </v-data-table>

            </v-card-text>

        </v-card>

    </v-flex>


</v-layout>

</v-container>

</template>

<script>

import accounting from 'accounting-js'
import moment from 'moment'
import lineChart from '@/components/line-chart'

export default {

    components: {lineChart}, 

    name: 'Main',

    data() {return {

        paymentMethod: 'avalanche',

        showInputError: false,
        showAddLoan: false,

        headers: [
            { text: 'Debt Name', value: 'name'},
            { text: 'Principal', value: 'principal', align: 'center'},
            { text: 'Interest Rate', value: 'interest_rate', align: 'center'},
            { text: 'Monthly Minimum Payment', value: 'monthtly_minimum', align: 'center'}
        ],

        input: {
            name: '',
            principal: '',
            interest_rate: '',
            monthtly_minimum: '',
            debt_color: ''
        },

        loans: [ ],

        chartData: {
            labels: ['Something'],
            datasets: [
                {
                    label: 'Debt',
                    backgroundColor: '#f87979',
                    data: [100]
                }
            ]
        }, 

    }},

    computed: {

        chartOptions(){
            return {
                responsive: true, 
                maintainAspectRatio: false,
                 scales: {
                    yAxes: [{
                        ticks: {
                            beginAtZero: true
                        }
                    }]
                }
            }
        },

        dates(){
            let temp = [];
            let date = moment();
            let principal = this.loans.reduce( (acc, curr) => acc+curr.principal , 0);
            let monthly_total_amt = this.loans.reduce( (acc, curr) => acc+curr.monthtly_minimum , 0);
            //let avgInterestRate = (this.loans.reduce( (acc,curr)=> acc+parseFloat(curr.interest_rate) ,0) / this.loans.length) / 100;
            while(principal > 0){
                //principal += principal * avgInterestRate; 
                principal = principal - monthly_total_amt;
                date.add(1, 'month');
                temp.push(date.format('MM/DD/YYYY'));
            }
            return temp
        },

        validateInputs() {
            return accounting.unformat(this.input.principal) != 0
                && accounting.unformat(this.input.monthtly_minimum) != 0
                && !isNaN(parseFloat(this.input.interest_rate))
        }

    },

    methods: {

        updateChart(){

            let datasets = [];

            this.loans.forEach( loan => {

                datasets.push({
                    label: loan.name,
                    backgroundColor: loan.debt_color,
                    data: this.getPayments(loan)
                });

            });

            this.chartData = { labels: this.dates, datasets }
        },

        getPayments(loan){
            let temp = [];
            let principal = 0+parseFloat(loan.principal);
            let monthly_total_amt = 0+parseFloat(loan.monthtly_minimum);
            let interest = 0+(parseFloat(loan.interest_rate)/100/12);
            while(principal > 0){
                principal += principal * interest;
                principal = principal - monthly_total_amt;
                temp.push( principal.toFixed(2) );
            }
            return temp;
        },

        toggleInputError(){
            this.showInputError = false;
        },

        cancelAddLoan(){
            this.showAddLoan = false;
            this.input = {
                name: '',
                principal: '',
                interest_rate: '',
                monthtly_minimum: ''
            };
        },

        addLoan() {

            if(this.validateInputs){
                this.loans.push(this.input);
                this.showAddLoan = false;
                this.updateChart();
            }else{
                this.showInputError = true;
            }

        }

    }

}

</script>

<style>

</style>
