<template>
  <div>
    <Panel>
      <div slot="title">{{$t('m.Problems_List')}}</div>
      <Table v-if="contestRuleType == 'ACM' || OIContestRealTimePermission"
             :columns="ACMTableColumns"
             :data="problems"
             @on-row-click="goContestProblem"
             no-data-text="No Problems"></Table>
      <Table v-else
             :data="problems"
             :columns="OITableColumns"
             @on-row-click="goContestProblem"
             no-data-text="No Problems"></Table>
    </Panel>
  </div>
</template>

<script>
  import {mapState, mapGetters} from 'vuex'
  import {ProblemMixin} from '@oj/components/mixins'
  import api from '@oj/api'
  import { USER_TYPE } from '@/utils/constants'
  export default {
    name: 'ContestProblemList',
    mixins: [ProblemMixin],
    data () {
      return {
        ACMTableColumns: [
          {
            title: '#',
            key: '_id',
            sortType: 'asc',
            width: 150
          },
          {
            title: 'Title',
            key: 'title'
          },
          {
            title: 'Total',
            key: 'submission_number'
          },
          {
            title: 'AC Rate',
            render: (h, params) => {
              return h('span', this.getACRate(params.row.accepted_number, params.row.submission_number))
            }
          }
        ],
        OITableColumns: [
          {
            title: '#',
            key: '_id',
            width: 150
          },
          {
            title: 'Title',
            key: 'title'
          }
        ],
        ContestProblemEnter_column: false,
        ContestProblemRejudge_column: false
      }
    },
    mounted () {
      this.getContestProblems()
    },
    methods: {
      getContestProblems () {
        this.$store.dispatch('getContestProblems').then(res => {
          if (this.isAuthenticated) {
            if (this.contestRuleType === 'ACM') {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            } else if (this.OIContestRealTimePermission) {
              this.addStatusColumn(this.ACMTableColumns, res.data.data)
            }
          }
          // this.adjustContestProblemEnterColumn()
          this.adjustContestProblemRejudgeColumn()
        })
      },
      goContestProblem (row) {
        this.$router.push({
          name: 'contest-problem-details',
          params: {
            contestID: this.$route.params.contestID,
            problemID: row._id
          }
        })
      },
      /*
      * Contest Problem Entrance @2018.11.19
      */
      adjustContestProblemEnterColumn () {
        if (this.ContestProblemEnter_column) {
          return
        }
        const EnterColumn = {
          title: 'Option',
          fixed: 'right',
          align: 'center',
          width: 120,
          render: (h, params) => {
            return h('Button', {
              props: {
                type: 'primary',
                size: 'small'
              },
              on: {
                click: () => {
                  this.loading = true
                  this.goContestProblem(params.row)
                  this.loading = false
                }
              }
            }, 'Enter')
          }
        }
        if (this.contestRuleType === 'ACM') {
          this.ACMTableColumns.push(EnterColumn)
        } else if (this.contestRuleType === 'OI') {
          this.OITableColumns.push(EnterColumn)
        }
        this.ContestProblemEnter_column = true
      },
      /*
      * Contest Problem Rejudge @2018.11.18
      * Simditor : v2.3.6
      * */
      adjustContestProblemRejudgeColumn () {
        if (this.ContestProblemRejudge_column || !this.ContestProblemRejudgeColumnVisible) {
          return
        }
        const RejudgeColumn = {
          title: 'Admin Option',
          fixed: 'right',
          align: 'center',
          width: 120,
          render: (h, params) => {
            return h('Button', {
              props: {
                type: 'primary',
                size: 'small'
              },
              on: {
                click: (event) => {
                  this.$Modal.confirm({
                    width: 350,
                    loading: false,
                    title: 'Rejudge Comfirm',
                    cancelText: 'Cancel',
                    okText: 'Rejudge',
                    content: 'Really want to Rejudge Problem?',
                    onOk: () => {
                      this.loading = true
                      this.handleContestProblemRejudge(params.row._id, this.$route.params.contestID)
                      this.loading = false
                    }
                  })
                  event.stopPropagation()
                }
              }
            }, 'Rejudge')
          }
        }
        if (this.contestRuleType === 'ACM') {
          this.ACMTableColumns.push(RejudgeColumn)
        } else if (this.contestRuleType === 'OI') {
          this.OITableColumns.push(RejudgeColumn)
        }
        this.ContestProblemRejudge_column = true
      },
      handleContestProblemRejudge (problemID, contestID) {
        api.problemRejudge(problemID, contestID).then(() => {
          this.$success('Rejudge Successed')
        }, () => {
        })
      }
    },
    computed: {
      ...mapState({
        problems: state => state.contest.contestProblems
      }),
      ...mapGetters(['isAuthenticated', 'contestRuleType', 'OIContestRealTimePermission', 'user']),
      /**
       * @return {boolean}
       */
      ContestProblemRejudgeColumnVisible () {
        return this.$route.params.contestID && this.user.admin_type === USER_TYPE.SUPER_ADMIN
      }
    }
  }
</script>

<style scoped lang="less">
</style>
