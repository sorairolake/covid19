<template>
  <v-col
    cols="12"
    :md="isSingleCard || 6"
    class="DataCard DeathsByDeathDateCard"
  >
    <client-only>
      <time-bar-chart
        :title="$t('死亡日別による死亡者数の推移')"
        :title-id="'deaths-by-death-date'"
        :chart-id="'deaths-by-death-date'"
        :chart-data="graphData"
        :date="date"
        :unit="$t('人')"
        :day-period="isSingleCard ? 120 : 60"
        :is-single-card="isSingleCard"
      >
        <template #additionalDescription>
          <span>{{ $t('（注）') }}</span>
          <ul>
            <li>
              {{
                $t(
                  '各保健所から報告のあった死亡者について、死亡日別に整理したものである'
                )
              }}
            </li>
            <i18n
              tag="li"
              path="死亡日不明者はグラフから除いているため、累計値が「{confirmed}」の死亡者数と一致しない"
            >
              <template #confirmed>
                <app-link
                  :to="localePath('/cards/details-of-confirmed-cases/')"
                >
                  {{ $t('検査陽性者の状況') }}
                </app-link>
              </template>
            </i18n>
          </ul>
        </template>
      </time-bar-chart>
    </client-only>
  </v-col>
</template>

<script>
import AppLink from '@/components/_shared/AppLink.vue'
import TimeBarChart from '@/components/index/_shared/TimeBarChart.vue'
import deaths from '@/data/deaths.json'
import calcDayBeforeRatio from '@/utils/calcDayBeforeRatio'
import formatGraph from '@/utils/formatGraph'
import { isSingleCard } from '@/utils/urls'

export default {
  components: {
    TimeBarChart: {
      extends: TimeBarChart,
      computed: {
        displayInfo() {
          const { lastDay, lastDayData } = calcDayBeforeRatio({
            displayData: this.displayData,
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
    AppLink,
  },

  data() {
    const formatData = deaths.data.map((data) => {
      return {
        日付: data.death_date,
        小計: data.count,
      }
    })

    // 死亡者数グラフ
    const graphData = formatGraph(formatData)

    const { date } = deaths

    return {
      date,
      graphData,
    }
  },
  computed: {
    isSingleCard() {
      return isSingleCard(this.$route.path)
    },
  },
}
</script>
