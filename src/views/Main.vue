<template>

<v-container grid-list-md fluid>

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

                    <v-text-field v-model="monthly_payment" placeholder="$50" solo />

                    <v-slide-y-reverse-transition>
                        <p class="red--text" v-show="!monthlyPaymentMeetsMinimum">Amount does not meet monthly minimum!</p>
                    </v-slide-y-reverse-transition>

                </v-form>


            </v-card-text>

        </v-card>

    </v-flex>

    <v-flex v-show="loans.length == 0" xs12 md9>

        <v-jumbotron height="100%" color="primary">
            <v-container fill-height>
                <v-layout align-center>
                    <v-flex>

                        <h3 class="display-2 white--text">IN DEBT? NEED A LIFE-RAFT?</h3>

                        <v-divider class="my-3 white white--text"></v-divider>

                        <span class="subheading white--text">Raft is a loan calculator that helps you pay off your debts.<br /><br />Discover an optimized payment plan to minimize interest and get debt free as fast as possible.</span>

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
                                        <v-text-field @keydown.enter="addLoan" @input="toggleInputError" v-model="input.monthtly_minimum" label="Monthly Minimum Payment ($)" placeholder="$1000.00" />

                                        <v-slide-y-transition> 
                                            <p v-show="showInputError" class="subheading red--text">Invalid Values!</p>
                                        </v-slide-y-transition>

                                        <v-slide-y-transition> 
                                            <p v-show="!loanValid && showInputError" class="subheading red--text">Unfeasible Loan! You're interest payments are higher than your monthly minimum!</p>
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

    <v-flex v-show="loans.length > 0" text-xs-center xs12 md9>

        <v-container fluid grid-list-sm>
            <v-layout row wrap>
 
                <v-flex d-flex xs12 sm3>
                    <v-card color="purple white--text">
                        <v-card-title class="subheading">
                            <v-layout>
                                <v-flex><v-icon color="white">fa-dollar-sign</v-icon></v-flex>
                                <v-flex>Principal Paid</v-flex>
                            </v-layout>
                        </v-card-title>
                        <v-card-text>
                            <h1 class="title">{{totalPrincipal | displayMoney}}</h1>
                            <p>That's only {{totalPrincipal | displayMoney}} to go!</p>
                        </v-card-text>
                    </v-card>
                </v-flex>

                <v-flex d-flex xs12 sm3>
                    <v-card color="teal lighten-2 white--text">
                        <v-card-title class="subheading">
                            <v-layout>
                                <v-flex><v-icon color="white">fa-calendar-alt</v-icon></v-flex>
                                <v-flex>Paid Off</v-flex>
                            </v-layout>
                        </v-card-title>
                        <v-card-text>
                            <h1 class="title">{{payoff_date}}</h1>
                            <p>Debt free in {{time_to_payoff}}</p>
                        </v-card-text>
                    </v-card>
                </v-flex>

                <v-flex d-flex xs12 sm3>
                    <v-card color="orange darken-2 white--text">
                        <v-card-title class="subheading">
                            <v-layout>
                                <v-flex><v-icon color="white">fa-calendar-alt</v-icon></v-flex>
                                <v-flex>Interest Paid</v-flex>
                            </v-layout>
                        </v-card-title>
                        <v-card-text>
                            <h1 class="title">{{total_interest_paid | displayMoney}}</h1>
                            <p>Oh No!</p>
                        </v-card-text>
                    </v-card>
                </v-flex>

                <v-flex d-flex xs12 sm3>
                    <v-card color="yellow darken-2 white--text">
                        <v-card-title class="subheading">
                            <v-layout>
                                <v-flex><v-icon color="white">fa-calendar-alt</v-icon></v-flex>
                                <v-flex>Average Interest Rate</v-flex>
                            </v-layout>
                        </v-card-title>
                        <v-card-text>
                            <h1 class="title">{{avg_interest_rate}}%</h1>
                            <p>Your average interest rate across all loans</p>
                        </v-card-text>
                    </v-card>
                </v-flex>

                <v-flex xs12>

                    <v-tabs color="primary" dark slider-color="secondary" >
                        <v-tab>Principal Remaining</v-tab>
                        <v-tab-item>
                            <v-card>

                                <v-card-title>

                                </v-card-title>

                                <v-card-text>
                                    <line-chart :chartData="chartData" :options="chartOptions"></line-chart>
                                </v-card-text>

                            </v-card>
                        </v-tab-item>
                    </v-tabs>

                </v-flex>

            </v-layout> 
        </v-container>

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

        monthly_payment: 0,
        numberOfPayments: 0,

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
            debt_color: '#f44242'
        },

        loans: [ ],

        payments: [],

        chartData: {
            labels: ['Something'],
            datasets: [
                {
                    label: 'Debt',
                    backgroundColor: 'rgba(200, 0, 0, 0.1)',
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

        avg_interest_rate(){
            return (this.loans.reduce( (acc,curr)=>acc+parseFloat(curr.interest_rate) ,0) / this.loans.length).toFixed(2);
        },

        loanValid(){
           return parseFloat(this.input.monthtly_minimum) > (parseFloat(this.input.interest_rate) / 100 /12) * parseFloat(this.input.principal)
        },

        validateInputs() {
            return accounting.unformat(this.input.principal) != 0
                && accounting.unformat(this.input.monthtly_minimum) != 0
                && !isNaN(parseFloat(this.input.interest_rate))
        },

        monthlyPaymentMeetsMinimum(){
            return this.monthly_payment >= this.loans.reduce( (acc,curr)=>acc+parseFloat(curr.monthtly_minimum) ,0);
        },

        totalPrincipal(){
            return this.loans.reduce( (acc,curr) => acc+parseFloat(curr.principal), 0)
        },

        payoff_date(){
            let date = moment();
            date.add(this.numberOfPayments, 'months');
            return date.format('MMMM YYYY')
        },

        time_to_payoff(){
            let date = moment();
            date.add(this.numberOfPayments, 'months');
            return date.fromNow();
        },

        total_interest_paid(){
            
            let total = 0;
            
            this.loans.forEach(loan => {
                let principal = 0+parseFloat(loan.principal);
                let monthtly_minimum = 0+parseFloat(loan.monthtly_minimum);
                let interest_rate = 0+(parseFloat(loan.interest_rate)/100/12);
                while(principal > 0){
                    let interest_paid = principal * interest_rate;
                    principal += interest_paid;
                    principal -= monthtly_minimum;
                    total += interest_paid;
                }
            });

            return total;

        }

    },

    methods: {

        updateChart(){

            let datasets = [];

            this.loans.forEach( loan => {

                datasets.push({
                    label: loan.name,
                    backgroundColor: loan.debt_color,
                    data: this.payments.filter(payment => payment.debt_name == loan.name).map(payment => payment.principal_remaining)
                });

            });

            this.chartData = { 
                labels: this.payments.filter(p => p.debt_name==this.loans[0].name).map(payment => payment.date), 
                datasets 
            }
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

        updateMinimumMonthlyPayment(){
            this.monthly_payment = this.loans.reduce( (acc,curr)=>acc+parseFloat(curr.monthtly_minimum) ,0)
        },

        createPayments(){

            this.payments = [];

            this.loans.forEach( loan => {

                let date = moment();

                let principal = parseFloat(loan.principal);
                let monthtly_minimum = parseFloat(loan.monthtly_minimum);
                let interest_rate = parseFloat(loan.interest_rate) / 100 / 12;

                let interest_paid = 0;

                while(principal > 0){

                    let interest = principal * interest_rate;

                    interest_paid += interest;
                    principal += interest;
                    principal -= monthtly_minimum;

                    if(principal <= 0){ principal = 0;}
                    this.payments.push({
                        debt_name: loan.name,
                        principal_remaining: parseFloat( principal.toFixed(2) ),
                        interest_paid: parseFloat( interest_paid.toFixed(2) ),
                        payment_amount: monthtly_minimum,
                        date: date.format('MM/YYYY')
                    });

                    date.add(1, 'month');

                }
                
            });



        },

        getRandom255Val(){
            return Math.floor( Math.random() * 255 )
        },

        getRandomTransparentColor(){
            return `rgba(${this.getRandom255Val()}, ${this.getRandom255Val()}, ${this.getRandom255Val()}, 0.3)`;
        },

        addLoan() {

            if(this.validateInputs && this.loanValid){
                this.input.debt_color = this.getRandomTransparentColor();
                this.loans.push(this.input);
                this.cancelAddLoan();
                this.updateMinimumMonthlyPayment();
                this.createPayments();
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
