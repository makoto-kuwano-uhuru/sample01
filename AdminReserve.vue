<template>
  <div class="l-page_content">
    <div class="l-page_section">
      <div class="l-page_section__inner">
        <h2 class="c-heading_lv2">予約一覧</h2>

        <div
          class="c-row u-flex u-align-items_flexend sp_flex-wrap_wrap u-sp_pr5p u-sp_pl5p"
        >
          <div class="c-cell u-mr10 u-sp_w100p">
            <dl>
              <dt class="u-bold u-font14">
                企業ID<b class="required">必須</b>
              </dt>
              <dd>
                <div class="c-input c-input--text--size_s">
                  <input
                    v-model="accountCd"
                    type="text"
                    class="text"
                    placeholder="IDを入力してください"
                  />
                </div>
              </dd>
            </dl>
          </div>
          <div class="c-cell u-mr10 u-sp_mr0 u-sp_w100p">
            <dl class="u-sp_flex sp_align-items_center">
              <dt class="u-bold u-font14 u-sp_w30p">部屋分割</dt>
              <dd class="sp_flex-1">
                <div class="c-input c-input--select">
                  <select
                    id="dividedList"
                    v-model="selectedDivided"
                    name=""
                    class="select"
                    v-on:change="changeList()"
                  >
                    <option value="" disabled selected style="display: none">
                      選択してください
                    </option>
                    <option value=" "></option>
                    <option
                      v-for="v in Object.values(divided)"
                      :value="v"
                      :key="v"
                    >
                      {{ v }}
                    </option>
                  </select>
                </div>
              </dd>
            </dl>
          </div>
          <div class="c-cell u-mr10 u-sp_mr0 u-sp_w100p u-sp_mb20">
            <dl class="u-sp_flex sp_align-items_center">
              <dt class="u-bold u-font14 u-sp_w30p">期間</dt>
              <dd class="sp_flex-1">
                <div class="u-flex">
                  <div
                    class="c-input c-input--select c-input--select--size_s u-sp_w48p tw-mr-2"
                  >
                    <select
                      v-model="selectedYear"
                      name=""
                      class="select"
                      id="yearList"
                      v-on:change="changeList()"
                    >
                      <option value="" disabled selected style="display: none">
                        年
                      </option>
                      <option value=" "></option>
                      <option v-for="y in selectableYear" :value="y" :key="y">
                        {{ y }}
                      </option>
                    </select>
                  </div>
                  <div
                    class="c-input c-input--select c-input--select--size_s u-sp_w48p"
                  >
                    <select
                      v-model="selectedMonth"
                      name=""
                      class="select"
                      id="monthList"
                      v-on:change="changeList()"
                    >
                      <option value="" disabled selected style="display: none">
                        月
                      </option>
                      <option value=" "></option>
                      <option v-for="m in selectableMonth" :value="m" :key="m">
                        {{ m }}
                      </option>
                    </select>
                  </div>
                </div>
              </dd>
            </dl>
          </div>
          <div class="c-cell u-mr10 u-sp_w100p">
            <dl>
              <dt class="u-bold u-font14">ルームNo</dt>
              <dd>
                <div class="c-input c-input--text--size_s">
                  <input
                    v-model="roomNo"
                    type="text"
                    class="text"
                    placeholder="401"
                  />
                </div>
              </dd>
            </dl>
          </div>
          <div class="c-cell u-sp_w100p">
            <div class="c-input c-input--btn_search">
              <input
                @click="onFilter"
                type="button"
                value="絞り込む"
                class="submit"
              />
            </div>
          </div>
        </div>

        <div
          v-if="isNoData || isRequired || systemError"
          class="c-row u-sp_pr5p u-sp_pl5p"
        >
          <div class="c-error">
            <p v-if="isNoData">該当する予約がありません。</p>
            <p v-if="isRequired">必須項目を入力してください。</p>
            <p v-if="systemError">
              エラーが発生しました。再度入力いただく、もしくは運営者へお問い合わせください。(エラーコード：{TBD})
            </p>
          </div>
        </div>

        <div class="c-table">
          <div class="c-table__inner">
            <table class="table">
              <thead class="thead">
                <tr class="tr">
                  <th class="th" width="80">&nbsp;</th>
                  <th class="th" width="80">ステータス</th>
                  <th class="th hidden-item" width="80">部屋分割</th>
                  <th class="th" width="120">予約情報変更締切</th>
                  <th class="th" width="120">試合日時</th>
                  <th class="th" width="140">対戦カード</th>
                  <th class="th" width="75">ルームNo</th>
                  <th class="th" width="110">人数</th>
                  <th class="th" width="115">企業名称</th>
                  <th class="th" width="210">予約管理者</th>
                </tr>
              </thead>
              <tbody class="tbody tw-overflow-y-scroll">
                <tr
                  class="tr"
                  v-for="reserve in dispReserves"
                  :key="reserve.id"
                >
                  <td class="td detail">
                    <div class="c-btn--black_mini">
                      <button
                        class="c-btn--black_mini__link"
                        :disabled="isDivide(reserve)"
                        @click="onDivied(reserve)"
                      >
                        部屋分割
                      </button>
                    </div>
                  </td>
                  <td class="td status">
                    <div
                      v-if="
                        isMatchCancelled(reserve) && !reserve.isDividedOrigin
                      "
                      class="c-label c-label--gray"
                    >
                      試合中止
                    </div>
                    <div
                      v-else-if="
                        isPastGame(reserve) && !reserve.isDividedOrigin
                      "
                      class="c-label c-label--gray"
                    >
                      利用済
                    </div>
                    <div
                      v-else-if="
                        overChangeLimit(reserve) && !reserve.isDividedOrigin
                      "
                      class="c-label c-label--blue"
                    >
                      締切済
                    </div>
                    <div
                      v-else-if="
                        registered(reserve) && !reserve.isDividedOrigin
                      "
                      class="c-label c-label--white"
                    >
                      登録済
                    </div>
                    <div
                      v-else-if="
                        unregistered(reserve) && !reserve.isDividedOrigin
                      "
                      class="c-label c-label--red"
                    >
                      未登録
                    </div>
                    <div
                      class="c-label tw-mt-2 tw-hidden sm:tw-block"
                      :class="{
                        'c-label--white': reserve.reserverId || reserve.divided,
                        'c-label--red': !reserve.reserverId && !reserve.divided,
                      }"
                    >
                      {{
                        reserve.reserverId || reserve.divided
                          ? '分割済'
                          : '未分割'
                      }}
                    </div>
                  </td>
                  <td class="td divided hidden-item">
                    <div
                      class="c-label"
                      :class="{
                        'c-label--white': reserve.reserverId || reserve.divided,
                        'c-label--red': !reserve.reserverId && !reserve.divided,
                      }"
                    >
                      {{
                        reserve.reserverId || reserve.divided
                          ? '分割済'
                          : '未分割'
                      }}
                    </div>
                  </td>
                  <td
                    class="td deadline"
                    :class="{
                      'sp-deadline':
                        unregistered(reserve) || registered(reserve),
                    }"
                  >
                    <span v-if="unregistered(reserve) || registered(reserve)"
                      >{{ getDayWithWeek(reserve.webReserveDeadLine)
                      }}<br class="sm:tw-hidden" />{{
                        getTime(reserve.webReserveDeadLine)
                      }}
                      まで</span
                    >
                  </td>
                  <td class="td date tw-font-bold">
                    <span
                      >{{ getDayWithWeek(reserve.gameAt)
                      }}<br class="sm:tw-hidden" />{{
                        getTime(reserve.gameStartTime)
                      }}</span
                    >
                  </td>
                  <td class="td card" style="text-align: center">
                    <span>
                      {{ getGameCard(reserve.gameCard, 0) }}
                      <br
                        class="sm:tw-hidden"
                        :class="{
                          'tw-hidden': !isExistStr(reserve.gameCard, 'VS'),
                        }"
                      />
                      <span
                        :class="{
                          'tw-hidden': !isExistStr(reserve.gameCard, 'VS'),
                        }"
                      >
                        VS
                      </span>
                      <br
                        class="sm:tw-hidden"
                        :class="{
                          'tw-hidden': !isExistStr(reserve.gameCard, 'VS'),
                        }"
                      />
                      {{ getGameCard(reserve.gameCard, 1) }}
                    </span>
                  </td>
                  <td class="td room_no tw-text-[#A48623]">
                    <span class="tw-text-[#2b241e]">{{
                      getRoomNo(reserve)
                    }}</span>
                  </td>
                  <td class="td person">
                    <!-- PCレイアウト -->
                    <dl class="dl-person sm:tw-hidden">
                      <dt v-if="countCheck(reserve.adultUsersCount)">大人</dt>
                      <dd
                        class="slash"
                        v-if="countCheck(reserve.adultUsersCount)"
                      >
                        /
                      </dd>
                      <dd
                        class="val"
                        v-if="countCheck(reserve.adultUsersCount)"
                      >
                        {{ reserve.adultUsersCount }}名
                      </dd>
                      <dt v-if="countCheck(reserve.minorUsersCount)">
                        4〜19歳
                      </dt>
                      <dd
                        class="slash"
                        v-if="countCheck(reserve.minorUsersCount)"
                      >
                        /
                      </dd>
                      <dd
                        class="val"
                        v-if="countCheck(reserve.minorUsersCount)"
                      >
                        {{ reserve.minorUsersCount }}名
                      </dd>
                      <dt v-if="countCheck(reserve.infantUsersCount)">
                        3歳以下
                      </dt>
                      <dd
                        class="slash"
                        v-if="countCheck(reserve.infantUsersCount)"
                      >
                        /
                      </dd>
                      <dd
                        class="val"
                        v-if="countCheck(reserve.infantUsersCount)"
                      >
                        {{ reserve.infantUsersCount }}名
                      </dd>
                    </dl>
                    <!-- スマホレイアウト -->
                    <dl class="dl-person tw-hidden sm:tw-block">
                      <dt>
                        <span v-if="countCheck(reserve.adultUsersCount)"
                          >大人</span
                        ><span v-if="countCheck(reserve.minorUsersCount)"
                          ><br
                            v-if="countCheck(reserve.adultUsersCount)"
                          />4〜19歳</span
                        ><span v-if="countCheck(reserve.infantUsersCount)"
                          ><br
                            v-if="
                              countCheck(reserve.adultUsersCount) ||
                              countCheck(reserve.minorUsersCount)
                            "
                          />3歳以下</span
                        >
                      </dt>
                      <dd class="slash">
                        <span v-if="countCheck(reserve.adultUsersCount)">/</span
                        ><span v-if="countCheck(reserve.minorUsersCount)"
                          ><br
                            v-if="countCheck(reserve.adultUsersCount)"
                          />/</span
                        ><span v-if="countCheck(reserve.infantUsersCount)"
                          ><br
                            v-if="
                              countCheck(reserve.adultUsersCount) ||
                              countCheck(reserve.minorUsersCount)
                            "
                          />/</span
                        >
                      </dd>
                      <dd class="val">
                        <span v-if="countCheck(reserve.adultUsersCount)"
                          >{{ reserve.adultUsersCount }}名</span
                        ><span v-if="countCheck(reserve.minorUsersCount)"
                          ><br v-if="countCheck(reserve.adultUsersCount)" />{{
                            reserve.minorUsersCount
                          }}名</span
                        ><span v-if="countCheck(reserve.infantUsersCount)"
                          ><br
                            v-if="
                              countCheck(reserve.adultUsersCount) ||
                              countCheck(reserve.minorUsersCount)
                            "
                          />{{ reserve.infantUsersCount }}名</span
                        >
                      </dd>
                    </dl>
                  </td>
                  <td class="td company_name">
                    <span>{{
                      reserve.contractedCompanyName?.companyName ?? ''
                    }}</span>
                  </td>
                  <td class="td reserver">
                    <span
                      >{{ reserve.reserver?.contactAccountName
                      }}<br class="sm:tw-hidden" />{{
                        reserve.reserver?.name
                          ? reserve.reserver?.name + ' 様'
                          : ''
                      }}
                      <br />{{
                        isReserverChangeExpired(
                          reserve.secretaryChangeExDateTime
                        )
                          ? '予約管理者変更期限切れ'
                          : ''
                      }}</span
                    >
                  </td>
                  <td class="td detail_sp">
                    <div class="c-btn--black_mini">
                      <button
                        class="c-btn--black_mini__link"
                        :disabled="isDivide(reserve)"
                        @click="onDivied(reserve)"
                      >
                        部屋分割
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
    <InfiniteLoading
      :identifier="dispReserves"
      :firstload="false"
      @infinite="load"
      class="infinite-load"
    >
      <template #complete class="tw-text-center">
        <span></span>
      </template>
    </InfiniteLoading>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'
import { API, Auth, graphqlOperation } from 'aws-amplify'
import dayjs from 'dayjs'
import ja from 'dayjs/locale/ja'
import type {
  ListReserveDetailsByCompanyIdQueryVariables,
  ListReserveDetailsByCompanyIdQuery,
  ModelSortDirection,
  ModelReserveFilterInput,
  ListAccountByCompanyCodeQueryVariables,
  ListAccountByCompanyCodeQuery,
  ModelAccountFilterInput,
} from '../../API'
import type { GraphQLQuery } from '@aws-amplify/api'
import {
  listAccountByCompanyCode,
  listReserveDetailsByCompanyId,
} from '../../graphql/customQueries'
import { useRoute, useRouter } from 'vue-router'
import InfiniteLoading from 'v3-infinite-loading'
import 'v3-infinite-loading/lib/style.css'
import { useReserveStore } from '../../stores/reserve'
import {
  filterNonNull,
  SCREEN,
  STATUS,
  STATUS_HIDE_CONDITION,
} from '../../helper/const'
import appLogger from '@/common/AppLogger'
import { isGraphQLError, wrapGraphQLError } from '@/helper/graphqlError'
import type { Unpacked } from '@/helper/types'
import type { CognitoUser } from '@aws-amplify/auth'
import { operationLogger } from '@/common/OperationLogger'
import { checkSessionWrap } from '@/middlewares/auth'

dayjs.locale(ja)

type Reservation = NonNullable<
  Unpacked<
    NonNullable<
      ListReserveDetailsByCompanyIdQuery['reservesByCompanyId']
    >['items']
  >
>

const { debug, info, error, warn } = appLogger()
const orgReserves = ref<Reservation[]>([])
const maxSize = ref<number>(0)
const pageNum = ref<number>(0)
const dispReserves = ref<Reservation[]>([])
const router = useRouter()
const route = useRoute()
const reserve_store = useReserveStore()
const preferredUsername = ref<string>() // [企業].[企業ID] or [企業担当者].[email]
const userId = ref<string>('')
const isCompanyOwner = ref<boolean>()
const dispNum = ref<number>(0)
const systemError = ref<boolean>(false)
const isNoData = ref<boolean>(false)
const isRequired = ref<boolean>(false)

// フィルター
const accountCd = ref('')
const selectedDivided = ref('')
const selectedYear = ref('')
const selectedMonth = ref('')
const roomNo = ref('')

const divided = {
  UNDIVIDED: '未分割',
  DIVIDED: '分割済',
} as const

const selectableYear = Array.from(
  Array(new Date().getFullYear() - 2022 + 1),
  (_, i) => i + 2022
)

const selectableMonth = Array.from(Array(12), (_, i) => i + 1)
// 画面サイズ
const mq = window.matchMedia('(min-width: 519px)')

const changeList = () => {
  placeHolderCheck()
}

watch(dispReserves, () => {
  if (!dispReserves.value || dispReserves.value.length == 0) {
    // 検索結果なし
    isNoData.value = true
  }
})

onMounted(async () => {
  // 条件分岐の判断に利用するメディアクエリを指定
  if (mq.matches) {
    dispNum.value = 10
  } else {
    dispNum.value = 5
  }
  updateSearchValues()
  await init()
  placeHolderCheck()
  emit('loadingEnd')
})
const emit = defineEmits(['loadingEnd'])

/** ブラウザバックなどの同ルート内で、クエリパラメータが変化した際に値を反映させる
 * ```/admin/reserve?accountCd=111 -> /admin/reserve?accountCd=222```
 */
watch(route, () => updateSearchValues())

/** クエリ文字列を検索条件に反映させる */
const updateSearchValues = () => {
  const { query } = route
  accountCd.value =
    !Array.isArray(query.accountCd) && query.accountCd != null
      ? query.accountCd
      : ''
  selectedDivided.value =
    !Array.isArray(query.selectedDivided) && query.selectedDivided != null
      ? query.selectedDivided
      : ''
  selectedYear.value =
    !Array.isArray(query.selectedYear) && query.selectedYear != null
      ? query.selectedYear
      : ''
  selectedMonth.value =
    !Array.isArray(query.selectedMonth) && query.selectedMonth != null
      ? query.selectedMonth
      : ''
  roomNo.value =
    !Array.isArray(query.roomNo) && query.roomNo != null ? query.roomNo : ''
}

/** 検索条件をクエリ文字列に反映させる */
const updateURLSearchParams = () => {
  const _accountCd = accountCd.value !== '' ? accountCd.value : undefined
  const _selectedDivided =
    selectedDivided.value !== '' && selectedDivided.value !== ' '
      ? selectedDivided.value
      : undefined
  const _selectedYear =
    selectedYear.value !== '' && selectedYear.value !== ' '
      ? selectedYear.value
      : undefined
  const _selectedMonth =
    selectedMonth.value !== '' && selectedMonth.value !== ' '
      ? selectedMonth.value
      : undefined
  const _roomNo = roomNo.value !== '' ? roomNo.value : undefined
  return router.replace({
    ...route,
    query: {
      ...route.query,
      accountCd: _accountCd,
      selectedDivided: _selectedDivided,
      selectedYear: _selectedYear,
      selectedMonth: _selectedMonth,
      roomNo: _roomNo,
    },
  })
}

/* 機能ID001 初期表示 */
const listReserveByCompanyIdQuery = async (
  params: ListReserveDetailsByCompanyIdQueryVariables
) =>
  API.graphql<GraphQLQuery<ListReserveDetailsByCompanyIdQuery>>(
    graphqlOperation(listReserveDetailsByCompanyId, params)
  )
const init = async () => {
  info('【FBサービス課予約一覧】初期表示処理 start')
  /* 予約（部屋）クエリ */
  isNoData.value = false
  try {
    // ログインユーザーのセッション情報を取得
    const cognitoUser: CognitoUser = await Auth.currentAuthenticatedUser()
    // TODO: 型エラーの削除（型: CognitoUserに「attributes」が含まれていなため型エラー）
    preferredUsername.value = cognitoUser?.attributes.preferred_username
    userId.value = ''
    // 監査ログ 作成
    operationLogger({
      eventName: '初期表示',
      screenName: SCREEN.ADMIN_RESERVE,
      userId: preferredUsername.value ?? undefined,
      createdAt: dayjs().format('YYYY-MM-DDTHH:mm:ss.SSS+09:00'),
    })
  } catch (e) {
    warn('【FBサービス課予約一覧】初期表示エラー')
    if (isGraphQLError(e)) warn(wrapGraphQLError(e))
    if (e instanceof Error) warn(e.message)
    systemError.value = true
  }
  info('【FBサービス課予約一覧】初期表示処理 end')
}

const getDayWithWeek = (date: string) => {
  const d = dayjs(date)
  return `${d.format('YYYY/MM/DD（ddd）')}`
}
const getTime = (date: string) => {
  return dayjs(date).format('HH:mm')
}
// 企業テーブル検索
const getCompany = async () => {
  const getCompanyList = (params: ListAccountByCompanyCodeQueryVariables) =>
    API.graphql<GraphQLQuery<ListAccountByCompanyCodeQuery>>(
      graphqlOperation(listAccountByCompanyCode, params)
    )
  let condition: {
    filter: ModelAccountFilterInput
    nextToken?: string | null
    limit?: number | null
  } = {
    filter: {
      and: [{ companyCode: { eq: accountCd.value } }],
    },
    limit: 1000,
  }
  let token: string | null = ''
  let isContinue = true
  let companyList = []
  // 検索
  do {
    // nextToken設定
    if (token != '') condition.nextToken = token
    // 検索
    let { data } = await getCompanyList(condition)
    // 検索結果格納
    if (data?.listAccounts?.items)
      companyList.push(...data?.listAccounts?.items.filter(filterNonNull))
    // nextToken取得
    token = data?.listAccounts?.nextToken ? data?.listAccounts?.nextToken : null
    // nextToken判定
    isContinue = token != null
  } while (isContinue)

  return companyList
}
const getReserveList = async () => {
  // 企業テーブル検索
  const accountList = await getCompany()
  if (!accountList.length) {
    // 該当する検索結果なし
    return []
  }
  const accountId = accountList[0].id
  const now = dayjs().format()
  // 予約検索
  let companyOwnerCondition: {
    contractedCompanyId: string
    sortDirection?: ModelSortDirection | null | undefined
    filter?: ModelReserveFilterInput | null | undefined
    limit?: number | null
    nextToken?: string | null
  } = {
    contractedCompanyId: accountId,
    filter: {
      and: [
        ...STATUS_HIDE_CONDITION,
        { isDeleted: { ne: true }, contactPersonViewFormula: { eq: true } },
      ],
    },
    limit: 1000,
  }
  let token: string | null = ''
  let isContinue = true
  let items = []
  // 検索
  do {
    // nextToken設定
    if (token != '') companyOwnerCondition.nextToken = token
    // 検索
    let { data } = await listReserveByCompanyIdQuery(companyOwnerCondition)
    // 検索結果格納
    if (data?.reservesByCompanyId?.items)
      items.push(...data?.reservesByCompanyId?.items)
    // nextToken取得
    token = data?.reservesByCompanyId?.nextToken
      ? data?.reservesByCompanyId?.nextToken
      : null
    // nextToken判定
    isContinue = token != null
  } while (isContinue)
  return items
}

/**
 * 絞り込みボタン押下
 */
const onFilter = checkSessionWrap(async () => {
  info('【FBサービス課予約一覧】絞り込み処理 start')
  isRequired.value = false
  isNoData.value = false
  systemError.value = false
  try {
    if (!accountCd.value) {
      // 企業ID入力なし
      isRequired.value = true
      return
    }
    updateURLSearchParams()
    const data = await getReserveList()
    orgReserves.value = data
      .filter(filterNonNull)
      .sort((a: Reservation, b: Reservation) => {
        // 試合日昇順
        if (a.gameStartTime !== b.gameStartTime) {
          return a.gameStartTime > b.gameStartTime ? 1 : -1
        }
        // ルームNo昇順
        if (a.roomNumber !== b.roomNumber) {
          return a.roomNumber > b.roomNumber ? 1 : -1
        }
        // グループ昇順
        let roomGroupA = a.roomGroup ? a.roomGroup : ''
        let roomGroupB = b.roomGroup ? b.roomGroup : ''
        return roomGroupA > roomGroupB ? 1 : -1
      })
  } catch (e) {
    warn('【FBサービス課予約一覧】絞り込みエラー')
    if (isGraphQLError(e)) warn(wrapGraphQLError(e))
    if (e instanceof Error) warn(e.message)
    systemError.value = true
    return
  }
  // 部屋分割フィルター
  switch (selectedDivided.value) {
    case divided.UNDIVIDED:
      orgReserves.value = orgReserves.value.filter(
        (v: Reservation) => !v.divided && !v.reserverId
      )
      break
    case divided.DIVIDED:
      orgReserves.value = orgReserves.value.filter(
        (v: Reservation) => v.divided || v.reserverId
      )
      break
    default:
      break
  }

  // 年フィルター
  if (selectedYear.value !== '' && selectedYear.value !== ' ') {
    orgReserves.value = orgReserves.value.filter((e: Reservation) => {
      return dayjs(e.gameAt).year() == Number(selectedYear.value)
    })
  }

  // 月フィルター
  if (selectedMonth.value !== '' && selectedMonth.value !== ' ') {
    if (selectedYear.value !== '' && selectedYear.value !== ' ') {
      orgReserves.value = orgReserves.value.filter((e: Reservation) => {
        return (
          dayjs(e.gameAt).year() === Number(selectedYear.value) &&
          dayjs(e.gameAt).month() === Number(selectedMonth.value) - 1
        )
      })
    } else {
      orgReserves.value = orgReserves.value.filter((e: Reservation) => {
        return dayjs(e.gameAt).month() === Number(selectedMonth.value) - 1
      })
    }
  }
  // ルームNoフィルター
  if (roomNo.value) {
    // 前方一致検索
    orgReserves.value = orgReserves.value.filter((e: Reservation) => {
      return e.roomNumber.startsWith(roomNo.value)
    })
  }
  // 表示用に格納
  // 絞り込み
  maxSize.value = orgReserves.value.length
  pageNum.value = 1 // 初期化ページ
  dispReserves.value = orgReserves.value.slice(0, dispNum.value)
  info('【FBサービス課予約一覧】絞り込み処理 end')
})
// 予約管理者変更有効期限切れ
const isReserverChangeExpired = (
  secretaryChangeExDateTime: string | null | undefined
) => {
  if (!secretaryChangeExDateTime) return false
  dayjs(secretaryChangeExDateTime)
  if (dayjs().isAfter(dayjs(secretaryChangeExDateTime))) {
    return true
  } else {
    return false
  }
}
// ステータス：試合中止判定
const isMatchCancelled = (reserve: Reservation) => {
  if (reserve.canceledCHK == true) {
    // 試合中止フラグがtrue
    return true
  } else {
    return false
  }
}
// ステータス：利用済
const pastArr: string[] = [
  STATUS.CHECKIN,
  STATUS.STARTUP,
  STATUS.LAUNCHED,
  STATUS.SETTLEMENT_START,
  STATUS.SETTLEMENT_END,
  STATUS.PAID,
]
// ステータス：利用済判定
const isPastGame = (reserve: Reservation) => {
  if (
    pastArr.indexOf(reserve.status ? reserve.status : '') !== -1 &&
    reserve.canceledCHK !== true
  ) {
    // 状況がリストのいずれか
    return true
  } else {
    return false
  }
}
// ステータス：締切済判定
const overChangeLimit = (reserve: Reservation) => {
  if (
    ((reserve.webRegistered && reserve.status === STATUS.DECISION) ||
      reserve.status === STATUS.BOOK) &&
    dayjs().isAfter(dayjs(reserve.webReserveDeadLine)) &&
    reserve.canceledCHK !== true
  ) {
    // (Web予約登録済チェックがfalseかつ状況が決定または本予約)かつ予約変更締切が現在日時以降
    return true
  } else {
    return false
  }
}
// ステータス：登録済判定
const registered = (reserve: Reservation) => {
  if (
    reserve.webRegistered &&
    reserve.status === STATUS.DECISION &&
    (dayjs().isBefore(dayjs(reserve.webReserveDeadLine)) ||
      dayjs().isSame(dayjs(reserve.webReserveDeadLine))) &&
    reserve.canceledCHK !== true
  ) {
    // 「Web予約登録済チェック === false」かつ「状況 === '決定'」かつ「現在日時 < 予約情報変更締切」
    return true
  } else {
    return false
  }
}
// ステータス：未登録判定
const unregistered = (reserve: Reservation) => {
  if (
    !reserve.webRegistered &&
    reserve.status === STATUS.BOOK &&
    (dayjs().isBefore(dayjs(reserve.webReserveDeadLine)) ||
      dayjs().isSame(dayjs(reserve.webReserveDeadLine))) &&
    reserve.canceledCHK !== true
  ) {
    // 「Web予約登録済チェック === false」かつ「状況 === '本予約'」かつ「現在日時 < 予約情報変更締切」
    return true
  } else {
    return false
  }
}

// 部屋分割可能チェック
const isDivide = (reserve: Reservation) => {
  if (
    (reserve.status === STATUS.BOOK || reserve.status === STATUS.DECISION) &&
    !isMatchCancelled(reserve) &&
    !dayjs().isAfter(dayjs(reserve.webReserveDeadLine))
  ) {
    /**
     * 状況が本予約、決定
     * 試合中止でない
     * 予約情報変更締切時間を過ぎていない
     */
    if (
      !(
        registered(reserve) &&
        reserve.reserverId &&
        !reserve.roomGroup &&
        !reserve.isDividedOrigin
      )
    ) {
      /**
       * 上記予約でも
       * 部屋分割「なし」かつ予約管理者設定有かつ予約情報登録済の予約は部屋分割不可
       * 条件：部屋分割できる条件
       * ステータスが登録済
       * 予約管理者IDが設定かつ
       * 部屋割りグループが未設定かつ
       * 部屋分割大元チェックがfalseでない予約
       */
      return false // 部屋分割可能
    }
  }
  return true // 部屋分割不可
}

// 対戦相手取得
const getGameCard = (gameCard: string, num: number) => {
  if (!gameCard) return ''
  return gameCard.split('VS')[num]?.trim()
}

const isExistStr = (str: string, searchStr: string) => {
  return str.indexOf(searchStr) != -1
}

// 部屋No
const getRoomNo = (reserve: Reservation) => {
  let roomNo = reserve.roomNumber
  if (reserve.roomGroup) {
    roomNo = roomNo + reserve.roomGroup
  }
  return roomNo
}

// 部屋分割
const onDivied = async (reserve: Reservation) => {
  // 監査ログ 作成
  operationLogger({
    eventName: `「部屋分割」ボタン押下: ${SCREEN.ADMIN_ROOMDIVISION_EDIT}遷移`,
    screenName: SCREEN.ADMIN_RESERVE,
    userId: preferredUsername.value ?? undefined,
    createdAt: dayjs().format('YYYY-MM-DDTHH:mm:ss.SSS+09:00'),
  })
  // 部屋分割編集admin-room-division
  router.push(`/admin/room-division/${reserve.id}`)
}
// ルームNo押下
const onRoom = async (reserve: Reservation) => {
  // 監査ログ 作成
  operationLogger({
    eventName: `「ルームNo」リンク押下: ${SCREEN.ROOMLAYOUT}遷移`,
    screenName: SCREEN.RESERVE,
    userId: preferredUsername.value ?? undefined,
    createdAt: dayjs().format('YYYY-MM-DDTHH:mm:ss.SSS+09:00'),
  })

  // ヘッダーの予約を選択中に変更
  reserve_store.selectReserve(reserve.id)
  router.push(`/reserve/${reserve.id}/layout?beforePage_reserve=true`)
}
// 予約管理者変更ボタン非活性
const isReserverChangeButtonDisabled = (reserve: Reservation) => {
  if (
    dayjs().isAfter(dayjs(reserve.webReserveDeadLine)) ||
    isMatchCancelled(reserve)
  ) {
    // 手仕舞い日時 < 現在日時 OR 試合中止
    return true
  }
  // 部屋分割大元の予約
  if (reserve.isDividedOrigin === true) return true
  if (isCompanyOwner.value === true) {
    // 窓口担当者の場合
    if (reserve.reserver && reserve.reserverId) {
      // 予約管理者が割り当てられている OR 試合中止
      return true
    }
  }
  return false
}
// 予約管理者変更
const onReserverChange = async (reserve: Reservation) => {
  // 監査ログ 作成
  operationLogger({
    eventName: `「予約管理者変更」ボタン押下: ${SCREEN.RESERVER_CHANGE}遷移`,
    screenName: SCREEN.RESERVE,
    userId: preferredUsername.value ?? undefined,
    createdAt: dayjs().format('YYYY-MM-DDTHH:mm:ss.SSS+09:00'),
  })

  // ヘッダーの予約を選択中に変更
  reserve_store.selectReserve(reserve.id)
  router.push(`/reserve/${reserve.id}/operator-change`)
}

// インフィニティスクロール
const load = async ($state: any) => {
  if (maxSize.value <= pageNum.value * dispNum.value) {
    debug('インフィニティスクロール終了')
    $state.complete()
    return
  }
  await new Promise((r) => setTimeout(r, 200))
  // 取得開始Index
  let getIndex = dispNum.value * pageNum.value
  // ページ追加
  pageNum.value = pageNum.value + 1
  // end取得個数
  let getNumber = dispNum.value * pageNum.value
  // 追加する予約
  let addReserves = orgReserves.value.slice(getIndex, getNumber)
  dispReserves.value.push(...addReserves)

  // インフィニティスクロール続行判定
  if (maxSize.value <= pageNum.value * dispNum.value) {
    $state.complete()
  } else {
    $state.loaded()
  }
}

// プレースホルダー対応
const placeHolderCheck = () => {
  // セレクトボックスプレースホルダー色変更
  var status = document.getElementById('statusList')
  if (accountCd.value == '') {
    if (status) status.style.color = '#808080'
  } else {
    if (status) status.style.color = '#2b241e'
  }
  var dividedEl = document.getElementById('dividedList')
  if (selectedDivided.value == '') {
    if (dividedEl) dividedEl.style.color = '#808080'
  } else {
    if (dividedEl) dividedEl.style.color = '#2b241e'
  }
  var year = document.getElementById('yearList')
  if (selectedYear.value == '') {
    if (year) year.style.color = '#808080'
  } else {
    if (year) year.style.color = '#2b241e'
  }
  var month = document.getElementById('monthList')
  if (selectedMonth.value == '') {
    if (month) month.style.color = '#808080'
  } else {
    if (month) month.style.color = '#2b241e'
  }
}

const countCheck = (count: number | null | undefined) => {
  if (count && count != 0) {
    return true
  } else {
    return false
  }
}
</script>
<style>
.infinite-load {
  margin: 10px;
  text-align: center;
  color: gray;
}
.dl-person > dt {
  width: 60px;
  float: left;
  text-align: left;
}
.dl-person > .slash {
  float: left;
}
.dl-person > .val {
  float: right;
}
.c-table .table .th {
  text-align: center;
}
@media screen and (max-width: 519px) {
  .dl-person > dt {
    width: 70px;
    float: left;
    text-align: left;
  }
  .dl-person > .val {
    float: right;
    text-align: right;
    width: 30px;
  }
  .c-table .table .th,
  .c-table .table .td {
    display: flex;
    align-items: flex-start;
    padding: 5px 10px;
  }
  .hidden-item {
    display: none !important;
  }
}
</style>
