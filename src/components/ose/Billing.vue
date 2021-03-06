<template>
    <div>
        <div class="hero is-light">
            <div class="hero-body">
                <div class="container">
                    <h1 class="title"><i class="material-icons">attach_money</i> Openshift-Kosten anzeigen</h1>
                </div>
                <h2 class="subtitle">
                    Hier findest du die Kosten für deine Openshift-Infrastrukturen</h2>
            </div>
        </div>
        <br>
        <form v-on:submit.prevent="getOpenshiftBilling">
            <b-message type="is-info">
                Alle Zahlen ohne Gewähr! Bitte verlassen sie sich auf die monatliche Abrechnung
            </b-message>
            <b-field :type="errors.has('ProjectContains') ? 'is-danger' : ''" label="Projekt"
                     :message="errors.first('ProjectContains')">
                <b-input v-model.trim="projectContains" name="ProjectContains"
                         ref="autofocus"
                         required
                         placeholder="projekt-%">
                </b-input>
            </b-field>

            <b-field label="Monat"
                     :type="errors.has('Monat') ? 'is-danger' : ''"
                     :message="errors.first('Monat')">

                <b-select placeholder="Wähle den Monat aus"
                          v-model="month"
                          required>
                    <option
                            v-for="month in months"
                            :value="month.value"
                            :key="month.key">
                        {{ month.key }}
                    </option>
                </b-select>
            </b-field>

            <label class="label">Cluster</label>
            <b-field>
                <b-radio-button v-model="cluster"
                                native-value="aws"
                                type="is-success">
                    <span>AWS</span>
                </b-radio-button>
                <b-radio-button v-model="cluster"
                                native-value="vis"
                                type="is-info">
                    <span>VIAS</span>
                </b-radio-button>
            </b-field>
            <button v-bind:class="{'is-loading': loading}"
                    :disabled="errors.any()"
                    class="button is-primary">
                Kostenberechnung erstellen
            </button>
        </form>
        <a v-if="csvDownload"
           class="button is-primary"
           :href="csvDownload"
           download="Openshift_Verrechnung.csv">Download CSV
        </a>
        <br>
        <b-table :data="data" narrowed detailed default-sort="Project">

            <template slot-scope="props">
                <b-table-column field="ReceptionAssignment" label="EmpfStelle" width="40" sortable>
                    {{ props.row.ReceptionAssignment }}
                </b-table-column>
                <b-table-column field="OrderReception" label="EmpfAuftrag" width="40" sortable>
                    {{ props.row.OrderReception }}
                </b-table-column>
                <b-table-column field="PspElement" label="Empfaenger-PSP-Element" width="40" sortable>
                    {{ props.row.PspElement }}
                </b-table-column>
                <b-table-column field="Project" label="Projekt" width="40" sortable>
                    {{ props.row.Project }}
                </b-table-column>
                <b-table-column field="Prices" label="Total" width="40" numeric sortable>
                    {{ getConsolidatedPrice(props.row.Prices) }} CHF
                </b-table-column>
            </template>
            <template slot="detail" slot-scope="props">
                <table>
                    <tr><td>
                        QuotaCpu:
                    </td><td>
                        {{ props.row.Prices.QuotaCpu }}
                    </td></tr>

                    <tr><td>
                        QuotaMemory:
                    </td><td>
                        {{ props.row.Prices.QuotaMemory }}
                    </td></tr>

                    <tr><td>
                        Storage:
                    </td><td>
                        {{ props.row.Prices.Storage }}
                    </td></tr>

                    <tr><td>
                        RequestedCpu:
                    </td><td>
                        {{ props.row.Prices.RequestedCpu }}
                    </td></tr>

                    <tr><td>
                        RequestedMemory:
                    </td><td>
                        {{ props.row.Prices.RequestedMemory }}
                    </td></tr>

                    <tr><td>
                        UsedCpu:
                    </td><td>
                        {{ props.row.Prices.UsedCpu }}
                    </td></tr>

                    <tr><td>
                        UsedMemory:
                    </td><td>
                        {{ props.row.Prices.UsedMemory }}
                    </td></tr>
                </table>
            </template>

            <div slot="empty" class="has-text-centered">
                Klicke auf den Button oben um die Kosten zu berechnen
            </div>
        </b-table>
    </div>
</template>

<script>
  export default {
    data() {
      return {
        data: [],
        months: [],
        month: 1,
        loading: false,
        csvDownload: '',
        cluster: 'aws',
        start: this.getStartDate(),
        end: this.getEndDate(),
        today: this.getToday(),
        projectContains: ''
      };
    },
    mounted: function () {
        this.getMonths(3);
    },
    methods: {
      getMonths: function(n) {
        var monthNames = ["Januar", "Februar", "März", "April", "Mai", "Juni",
            "Juli", "August", "September", "Oktober", "November", "Dezember"
        ];
        var today = new Date();
        var year = today.getFullYear();
        var month = today.getMonth();

        var i = 0;
        do {
            this.months.push({value: i, key: monthNames[month]});
            if(month == 1) {
                month = 12;
                year--;
            } else {
                month--;
            }
            i++;
        } while(i < n);
      },
      getOpenshiftBilling: function() {
        this.loading = true;
        var date = new Date();
        date.setDate(1);
        date.setMonth(date.getMonth()-this.month);
        this.$http.post(this.$store.state.backendURL + '/api/ose/chargeback', {
          date: date,
          cluster: this.cluster,
          projectContains: this.projectContains
        }).then((res) => {
          this.data = res.body.rows;
          let blob = new Blob([res.body.csv], {type: 'text/csv'});

          this.csvDownload = window.URL.createObjectURL(blob);
          this.loading = false;
        }, () => {
          this.loading = false;
        });
      },
      getConsolidatedPrice: function(prices) {
        var values = Object.keys(prices).map(function(e) {
            return prices[e]
        });
        return values.reduce((a,b) => a+b, 0);
      },
      getStartDate: function() {
        var date = new Date();
        return new Date(date.getFullYear(), date.getMonth()-1, 1);
      },
      getEndDate: function() {
        var date = new Date();
        return new Date(date.getFullYear(), date.getMonth(), 0);
      },
      getToday: function() {
        var date = new Date();
        return new Date(date.getFullYear(), date.getMonth(), date.getDate());
      }
    }
  };
</script>
