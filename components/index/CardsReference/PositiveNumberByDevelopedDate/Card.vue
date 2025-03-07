<template>
  <v-col cols="12" md="6" class="DataCard PositiveNumberByDevelopedDateCard">
    <client-only>
      <time-bar-chart
        :title="$t('発症日別による陽性者数の推移')"
        :title-id="'positive-number-by-developed-date'"
        :chart-id="'positive-number-by-developed-date'"
        :chart-data="graphData"
        :date="positiveByDeveloped.date"
        :unit="$t('人')"
        :url="'https://catalog.data.metro.tokyo.lg.jp/dataset/t000010d0000000093'"
      >
        <template #additionalDescription>
          <span>{{ $t('（注）') }}</span>
          <ul>
            <li>
              {{
                $t(
                  '各保健所から報告があった患者の発生情報を、発症日別に整理したものである'
                )
              }}
            </li>
            <li>
              {{
                $t(
                  '発症日不明者（検査結果が陽性であっても症状がない、発症日を覚えていないなど）はグラフから除いている'
                )
              }}
            </li>
            <li>
              {{
                $t(
                  '2022年2月2日以降の報告分には、感染者の濃厚接触者が有症状となった場合で、検査を実施せずに医師の判断により臨床診断された患者を含む'
                )
              }}
            </li>
          </ul>
        </template>
        <template #additionalInfoPanel>
          <data-view-custom-info-panel
            :l-text="lText"
            :s-text="sText"
            :num="num"
          />
        </template>
      </time-bar-chart>
    </client-only>
  </v-col>
</template>

<script>
import TimeBarChart from '@/components/index/_shared/TimeBarChart.vue'
import DataViewCustomInfoPanel from '@/components/index/CardsReference/PositiveNumberByDevelopedDate/DataViewCustomInfoPanel.vue'
import positiveByDeveloped from '@/data/positive_by_developed.json'
import calcDayBeforeRatio from '@/utils/calcDayBeforeRatio'
import formatGraph from '@/utils/formatGraph'
import { getCommaSeparatedNumberToFixedFunction } from '~/utils/monitoringStatusValueFormatters'

export default {
  components: {
    TimeBarChart: {
      extends: TimeBarChart,
      computed: {
        displayInfo() {
          const { lastDay, lastDayData } = calcDayBeforeRatio({
            displayData: this.displayData,
            dataIndex: 1,
          })
          const formattedLastDay = this.$d(lastDay, 'date')
          if (this.dataKind === 'transition') {
            return {
              lText: lastDayData,
              sText: `${formattedLastDay} ${this.$t('日別値')}（${this.$t(
                '現在判明している人数であり、後日修正される場合がある'
              )}）`,
              unit: this.unit,
            }
          }
          return {
            lText: lastDayData,
            sText: `${formattedLastDay} ${this.$t('累計値')}`,
            unit: this.unit,
          }
        },
      },
    },
    DataViewCustomInfoPanel,
  },
  data() {
    const formatData = positiveByDeveloped.data.map((data) => {
      return {
        日付: data.developed_date,
        小計: data.count,
      }
    })

    // 陽性患者数グラフ
    const graphData = formatGraph(formatData)

    // 発症日不明者数
    const formatter = getCommaSeparatedNumberToFixedFunction()
    const unknownCount = formatter(positiveByDeveloped.unknown_count)
    const lastDay = this.$d(
      new Date(positiveByDeveloped.data.slice(-1)[0].developed_date),
      'date'
    )

    return {
      positiveByDeveloped,
      graphData,
      lText: this.$t('発症日不明者 {num} 人を除く'),
      sText: `${lastDay} ${this.$t('累計値')}`,
      num: unknownCount,
    }
  },
}
</script>
