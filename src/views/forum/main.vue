<template>
<div>

<div v-if="false" class="ic-paper board-header" :style="lineStyleBG(board.id)" style="margin-bottom: 30px; margin-top: -15px; width: 100%">
    <div class="left">
        <h3 class="name">{{ board.name }}</h3>
        <div class="brief">{{ board.brief }}</div>
    </div>
</div>

<!-- 刷新标题 -->
<template v-if="board">
    <div v-title-dynamic v-if="$route.params.page && $route.params.page > 1">{{ board.name }} - 第{{$route.params.page}}页 - {{state.config.title}}</div>
    <div v-title-dynamic v-else>{{ board.name }} - {{state.config.title}}</div>
</template>
<div v-else v-title>全部主题 - {{state.config.title}}</div>

<div class="ic-container forum-box">
    <div class="wrapper">
        <div v-responsive.xs>
            <i @click="showSlideMenu = !showSlideMenu" class="icarus icon-comment-multiple-out" style="position: fixed; top: 6px; left: 20px; font-size: 22px; z-index: 1; color: #777" />

            <!-- xs 时的边栏 -->
            <transition enter-active-class="animated fadeIn" leave-active-class="animated fadeOut">
                <div v-if="showSlideMenu" class="dialog-overlay" @click="showSlideMenu = false"></div>
            </transition>

            <transition enter-active-class="animated slideInLeft faster" leave-active-class="animated slideOutLeft">
                <div v-if="showSlideMenu" class="left-nav bs4-xs">
                    <div class="left-nav-box">
                        <router-link class="ic-btn primary post-new-topic" @mouseover.native="mouseOverPostNewBtn = true" @mouseleave.native="mouseOverPostNewBtn = false" :style="postNewTopicStyle" :to="{ name: 'forum_topic_new', params: {'board_id': boardId } }">发表主题</router-link>

                        <div class="ul-boards">
                            <router-link :to="{ name: 'index', query: $route.query}" class="item" :class="{'showAll': !isBoard}" style="margin-top: 22px">
                                <div class="sign"></div>
                                <span class="title">全部主题</span>
                            </router-link>

                            <label v-if="isBoard" class="with-subboard-topic">
                                <ic-checkbox :size="14" v-model="withSubBoardTopic">包含子板块内容</ic-checkbox>
                            </label>
                            <div style="margin-bottom: 3px;"></div>
                            <router-link v-for="j in dymBoardList" :key="j.id" :class="{'subboard': j.parent_id}" class="item" :to="{ name: 'forum_board', params: {id: j.id}, query: $route.query }">
                                <div v-if="j.parent_id === null" class="sign" :style="lineStyleBG(j.id)"></div>
                                <span class="title" :style="boardNavStyle(j)">{{boardNavTitle(j)}}</span>
                            </router-link>
                        </div>
                    </div>
                </div>
            </transition>
        </div>

        <div class="left-nav ic-xs-hidden">
            <div class="left-nav-box">
                <!-- <span class="post-new-topic">板块列表</span> -->
                <router-link class="ic-btn primary post-new-topic" @mouseover.native="mouseOverPostNewBtn = true" @mouseleave.native="mouseOverPostNewBtn = false" :style="postNewTopicStyle" :to="{ name: 'forum_topic_new', params: {'board_id': boardId } }">发表主题</router-link>

                <div class="ul-boards">
                    <router-link :to="{ name: 'index', query: $route.query}" class="item" :class="{'showAll': !isBoard}" style="margin-top: 22px">
                        <div class="sign"></div>
                        <span class="title">全部主题</span>
                    </router-link>

                    <label v-if="isBoard" class="with-subboard-topic">
                        <ic-checkbox :size="14" v-model="withSubBoardTopic">包含子板块内容</ic-checkbox>
                    </label>
                    <div style="margin-bottom: 3px;"></div>
                    <router-link v-for="j in dymBoardList" :key="j.id" :class="{'subboard': j.parent_id}" class="item" :to="{ name: 'forum_board', params: {id: j.id}, query: $route.query }">
                        <div v-if="j.parent_id === null" class="sign" :style="lineStyleBG(j.id)"></div>
                        <span class="title" :style="boardNavStyle(j)">{{boardNavTitle(j)}}</span>
                    </router-link>
                </div>
            </div>
        </div>

        <div class="right" id="board-list">
            <top-btns :board="board"></top-btns>
            <div style="flex: 1 0 0%;">
                <!-- 加载占位条目 -->
                <template v-if="loading">
                    <div class="board-item-box" v-for="i in placeholderCount" :key="i">
                        <a class="board-item">
                            <div class="title-recent">
                                <avatar :size="32" :placeholder="true" class="avatar"></avatar>
                                <div class="right">
                                    <h2>
                                        <span class="placeholder-text f18"></span>
                                    </h2>
                                    <p class="topic-info" style="width: 50%">
                                        <span class="placeholder-text f14"></span>
                                    </p>
                                </div>
                                <div class="append-icons"></div>
                            </div>
                            <div class="detail ic-xs-hidden">
                                <div class="count-block">
                                    <div class="count">
                                        <p class="num"><span class="placeholder-text f16" style="width: 40%"></span></p>
                                        <p class="txt"><span class="placeholder-text f12" style="width: 30%"></span></p>
                                    </div>
                                    <div class="count">
                                        <p class="num"><span class="placeholder-text f16" style="width: 40%"></span></p>
                                        <p class="txt"><span class="placeholder-text f12" style="width: 30%"></span></p>
                                    </div>
                                </div>
                                <div class="recent ic-xs-hidden ic-sm-hidden ic-md-hidden">
                                    <span class="line"></span>
                                    <div class="post">
                                        <span class="placeholder-text f12" style="width: 50%"></span>
                                        <span class="placeholder-text f12"></span>
                                        <span class="placeholder-text f12" style="width: 75%"></span>
                                    </div>
                                    <div class="time">N/A</div>
                                </div>
                            </div>
                        </a>
                    </div>
                </template>

                <!-- 实际渲染条目 -->
                <template v-else-if="topics.items.length">
                    <div class="board-item-box" :key="i.id" v-for="i in topics.items"  @mouseover="itemHover(i.id)" @mouseout="itemHover(null)">
                        <router-link :to="{ name: 'forum_topic', params: {id: i.id} }" class="board-item" :class="{'top-post': i.sticky_weight}">
                            <div class="title-recent">
                                <avatar :user="i.user_id" :size="32" class="avatar" />
                                <div class="right">
                                    <h2>
                                        <router-link :title="i.title" :to="{ name: 'forum_topic', params: {id: i.id} }">
                                            <span>{{i.title}}</span>
                                            <span v-if="i.state === state.misc.POST_STATE.CLOSE">[关闭]</span>
                                        </router-link>
                                    </h2>

                                    <p class="topic-info">
                                        <router-link class="board-badge" :to="{ name: 'forum_board', params: {id: i.board_id} }">
                                            <div :style="lineStyleBG(i.board_id)" class="sign"></div>
                                            <span v-responsive.class class="name limit l2">{{boardBadgeTitleById(i.board_id)}}</span>
                                        </router-link>
                                        <user-link v-responsive.class class="author limit l2" :user="i.user_id" />
                                        <span class="time"><ic-time :timestamp="i.edit_time || i.time"/></span>
                                    </p>
                                </div>

                                <span class="icons">
                                    <i v-if="i.awesome == 1" class="awesome icarus icon-diamond" title="优秀" @click.prevent></i>
                                    <i v-if="false" class="icarus icon-crown" title="精华" style="color: #e8a85d"></i>
                                    <i v-if="isAdmin() && i.id === hoverId" class="manage icarus icon-39 animated rotateIn" title="管理" @click.prevent="setTopicManage(i)"></i>
                                </span>

                                <div class="append-icons">
                                    <i v-if="i.sticky_weight" class="icarus icon-pin" title="置顶" />
                                </div>
                            </div>
                            <div class="detail ic-xs-hidden">
                                <div class="count-block">
                                    <div class="count">
                                        <p class="num">{{i.s.click_count}}</p>
                                        <p class="txt">点击</p>
                                    </div>
                                    <div class="count">
                                        <p class="num">{{i.s.comment_count}}</p>
                                        <p class="txt">回复</p>
                                    </div>
                                </div>
                                <div class="recent ic-xs-hidden ic-sm-hidden ic-md-hidden">
                                    <span class="line" :style="lineStyle(i.board_id)"></span>
                                    <div class="post" v-if="i.s.last_comment_id && i.s.last_comment_id.id">
                                        <strong><i class="icon icarus icon-reply"></i><user-link :user="i.s.last_comment_id.user_id" />:</strong>
                                        <router-link tag="div" class="post-content" :to="{ name: 'forum_topic', params: {id: i.s.last_comment_id.related_id} }">{{atConvert(i.s.last_comment_id.content)}}</router-link>
                                    </div>
                                    <div class="post" v-else>无回复</div>
                                    <ic-time v-if="i.s.last_comment_id" class="time" :timestamp="i.s.last_comment_id.time" />
                                    <div v-else class="time">N/A</div>
                                </div>
                            </div>
                        </router-link>
                    </div>
                    <paginator v-if="isBoard" :page-info='topics' :route-name='"forum_board"' />
                    <paginator v-else :page-info='topics' :route-name='"forum_main"' />
                </template>

                <template v-else>
                    <div style="margin-left: 10px;">尚未有人发言……</div>
                </template>

                <!-- <div style="flex: 1 0 0%" v-if="(!loading) && (!(topics && topics.items.length))"> -->
            </div>
        </div>
    </div>
    <dialog-site-new v-if="isNewSite" />
    <dialog-user-set-nickname v-else-if="isNewUser"/>
    <dialog-topic-manage />
    <dialog-user-inactive-warn />
</div>
</div>
</template>

<style scoped lang="scss">
.placeholder-text {
    display: inline-block;
    background-color: #e9e9e9;
    width: 100%;

    &.f12 { height: 12px; }
    &.f14 { height: 14px; }
    &.f16 { height: 16px; }
    &.f18 { height: 18px; }
}

.wrapper {
    display: flex;

    .left-nav {
        flex: 5 1 0%;
        max-width: $page-left-max-width;
    }

    .left-nav.bs4-xs {
        position: fixed;
        height: 100%;
        top: 0;
        left: 0;
        background: #fff;
        z-index: 3;
        padding-left: 15px;
        padding-top: 30px;
        width: 180px;
    }

    .right {
        flex: 19 1 0%;
    }
}

.dialog-overlay {
    top: 0;
    z-index: 2;
}

$left-nav-padding-right: 30px;
$left-nav-sign-padding: 5px;

.ul-boards {
    margin: 0;
    list-style: none;

    .item {
        display: flex;
        align-items: center;
        padding: 7px 0;
        font-size: 14px;
        font-weight: bolder;
        color: lighten($gray-600, 10%);

        &.subboard {
            padding-left: 16px;
        }

        &.showAll {
            font-weight: bold;
            color: $primary;
        }

        .sign {
            width: 1em;
            height: 1em;
            background-color: #000;
            border-radius: 3px;
            flex-shrink: 0;
        }

        .title {
            margin-left: $left-nav-sign-padding;
        }
    }
}

.left-nav-box {
    padding: 0 $left-nav-padding-right 0 0;
}

.post-new-topic {
    width: 100%;
    display: block;
}

.with-subboard-topic {
    display: flex;
    align-items: center;
    font-size: 14px;
    user-select: none;
    margin-bottom: 7px;
    color: $gray-600;

    > .ic-checkbox {
        width: 100%;
    }

    span {
        margin-left: $left-nav-sign-padding;
    }
}

.board-header {
    background: #1f8dd6;
    padding: 2em 1em;
    color: #fff;
    display: flex;
    justify-content: center;
    text-align: center;
}
</style>

<script>
import api from '@/netapi.js'
import state from '@/state.js'
import '@/assets/css/_forum.scss'
import TopBtns from './topbtns.vue'
import ZingTouch from 'zingtouch'
import nprogress from 'nprogress/nprogress.js'

let pageOneHack = function (to, from, next) {
    // 这一hack的目标是抹除 /r/1 的存在，使其与 / 看起来完全一致
    // 但似乎由于 nprogress 的存在，显得有点僵硬
    if (to.name === 'forum_main' && (to.params.page === '1' || to.params.page === 1)) {
        if (from.name === 'index') {
            state.loading = 0
            nprogress.done()
            return next(false)
        }
        return next({ name: 'index', query: to.query })
    }
    next()
}

export default {
    data () {
        return {
            state,
            hoverId: null,
            loading: true,
            topics: { items: [] },
            withSubBoardTopic: false,
            withSubBoardTopicOptionReady: false,
            mouseOverPostNewBtn: false,
            placeholderCount: 20,
            // xs大小时的边栏
            showSlideMenu: false
        }
    },
    computed: {
        isNewSite: function () {
            if (this.state.boards.rawLst.length === 0) {
                return true
            }
        },
        isNewUser: function () {
            if (!state.user) return
            let ts = new Date().getTime() / 1000
            if ((state.user.is_new_user) && (ts - state.user.time < 24 * 60 * 60)) {
                state.dialog.userSetNickname = true
                return true
            }
        },
        postNewTopicStyle: function () {
            let exInfo = $.getBoardExInfoById(this.boardId)
            if (this.isBoard && (!this.state.boards.loaded)) {
                return { 'background-color': '#777' }
            }
            if (exInfo) {
                if (this.mouseOverPostNewBtn) {
                    return { 'background-color': exInfo.colorHover }
                }
                return { 'background-color': exInfo.color }
            }
        },
        isBoard: function () {
            return this.$route.name === 'forum_board'
        },
        board: function () {
            let bid = this.boardId
            if (bid) return $.getBoardInfoById(bid)
        },
        boardId: function () {
            if (this.isBoard) {
                return this.$route.params.id
            }
        },
        dymBoardList: function () {
            let lst = []
            let curBoardId = this.boardId
            let chain = $.getBoardChainById(curBoardId)
            let topBoardId = chain[chain.length - 1]

            let pushSubBoards = (i, chainIndex) => {
                let topId = chain[--chainIndex]
                // $.getBoardExInfoById(i.id)
                for (let j of this.state.boards.exInfoMap[i.id].subboards) {
                    lst.push(j)
                    if (j.id === topId) {
                        pushSubBoards(j, chainIndex)
                    }
                }
            }

            for (let i of this.state.boards.lst) {
                lst.push(i)

                if (i.id === topBoardId) {
                    pushSubBoards(i, chain.length - 1)
                }
            }

            return lst
        }
    },
    methods: {
        atConvert: $.atConvert2,
        isAdmin: function () {
            return $.isAdmin()
        },
        boardBadgeTitleById: function (id) {
            let chain = $.getBoardChainById(id)
            let ret = ''
            if (chain.length > 1) chain = chain.slice(0, -1)
            for (let i = chain.length - 1; i >= 0; i--) {
                let board = this.getBoardInfo(chain[i])
                ret += `${board.name}`
                if (i !== 0) ret += '/'
            }
            return ret
        },
        boardNavTitle: function (board) {
            if (board.parent_id) {
                let chain = $.getBoardChainById(board.id)
                let prefix = _.times(chain.length - 2, () => '>').join('')
                return `${prefix} ${board.name}`
            }
            return board.name
        },
        boardNavStyle: function (board) {
            if (this.isBoard) {
                let chain = $.getBoardChainById(this.boardId)
                if (chain.indexOf(board.id) !== -1) {
                    let exInfo = $.getBoardExInfoById(board.id)
                    return {
                        'color': exInfo.color,
                        'font-weight': 'bold'
                    }
                }

                // 如果上一级被选中且开启了“包含子版块内容”
                if (this.withSubBoardTopic && board.parent_id === this.boardId) {
                    // let exInfo = $.getBoardExInfoById(board.id)
                    // return {'color': exInfo.color}
                    // 本来是各种颜色，但是花花绿绿怪怪的，改成一个色
                    return { 'color': 'rgb(126, 140, 201)' }
                }
            }
        },
        setTopicManage: function (topic) {
            state.dialog.topicManageData = topic
            state.dialog.topicManage = true
        },
        itemHover: function (id) {
            this.hoverId = id
        },
        lineStyle: function (boardId, key = 'border-left-color') {
            return $.lineStyleById(boardId, key)
        },
        lineStyleBG: function (boardId) {
            return $.lineStyleById(boardId, 'background-color')
        },
        getBoardInfo: function (boardId) {
            return $.getBoardInfoById(boardId)
        },
        fetchData: async function () {
            this.$set(this, 'loading', true)
            this.loading = true
            let baseQuery1 = {
                select: 'id, time, edit_time, user_id, board_id, title, state, awesome, weight, update_time, sticky_weight',
                loadfk: { 'user_id': null, 'id': { 'as': 's', loadfk: { 'last_comment_id': { 'loadfk': { 'user_id': null } } } } }
            }
            let baseQuery = _.cloneDeep(baseQuery1)
            let params = this.$route.params
            let page = 1

            // 包含子板块内容
            if (localStorage.getItem('sbt')) {
                this.withSubBoardTopic = true
            }
            this.$nextTick(() => {
                this.withSubBoardTopicOptionReady = true
            })

            // 获取全局板块信息
            await $.getBoardsInfo()

            if (this.isNewSite) {
                state.dialog.siteNew = true
            }

            // 具体板块
            if (this.isBoard) {
                // 若是要求包含子板块内容
                if (localStorage.getItem('sbt')) {
                    let lst = [params.id]
                    for (let i of $.getBoardExInfoById(params.id).subboardsAll) {
                        lst.push(i.id)
                    }
                    baseQuery['board_id.in'] = JSON.stringify(lst)
                } else {
                    baseQuery['board_id'] = params.id
                }
                page = params.page
            } else {
                baseQuery['sticky_weight.ne'] = 5
                page = params.page
            }

            let query = this.$route.query
            let order = 'weight.desc, update_time.desc' // 权重降序

            if (query.type === '2' || query.type === 2) {
                // 最近更新：更新时间降序
                order = 'update_time.desc, time.desc'
            }

            if (query.type === '3' || query.type === 3) {
                // 最近发布：发布时间降序
                order = 'time.desc'
            }

            if (query.type === '4' || query.type === 4) {
                // 最近回复：回复时间排序
                // 好吧，这个好像暂时还实现不了
                order = 'update_time.desc, time.desc'
            }

            if (this.isBoard) {
                // 在板块模式下加入当前板块的置顶
                order = 'sticky_weight.desc, ' + order
            }

            let retList = await api.topic.list(Object.assign({
                order: order
            }, baseQuery), page)
            if (retList.code === api.retcode.SUCCESS) {
                if (!this.isBoard && (!page || page === 1)) {
                    // 首页
                    let retStickyTopics = await api.topic.list(Object.assign({
                        sticky_weight: 5, // 全局置顶项
                        order: order
                    }, baseQuery1))
                    if (retStickyTopics.code === api.retcode.SUCCESS) {
                        retList.data.items = _.concat(retStickyTopics.data.items, retList.data.items)
                    }
                }

                this.topics = retList.data
                this.loading = false
                return
            } else {
                this.topics = { items: [] }
            }

            // $.message_by_code(retList.code)
            this.loading = false
        }
    },
    beforeRouteEnter (to, from, next) {
        return pageOneHack(to, from, next)
    },
    beforeRouteUpdate (to, from, next) {
        return pageOneHack(to, from, next)
    },
    beforeRouteLeave (to, from, next) {
        if (to.name === 'forum_topic_new' && this.state.isInactiveUser()) {
            state.dialog.userInactive = true
            return false
        }
        return next()
    },
    created () {
        this.fetchData()

        this.$nextTick(() => {
            state.zt = state.zt || new ZingTouch.Region(document.body, false, false)
            let el = document.querySelector('.main')
            if (!el) return
            state.zt.unbind(el, 'swipe')
            state.zt.bind(el, 'swipe', (e) => {
                let info = e.detail.data[0]
                let d = info.currentDirection
                if (d < 50 || d > 310) {
                    // 向右滑动
                    this.showSlideMenu = true
                } else if (d > 130 && d < 230) {
                    // 向左滑动
                    this.showSlideMenu = false
                }
            }, false)
        })
    },
    watch: {
        // 如果路由有变化，会再次执行该方法
        '$route': 'fetchData',
        'withSubBoardTopic': async function (newVal, oldVal) {
            if (this.withSubBoardTopicOptionReady) {
                localStorage.removeItem('sbt', newVal)
                if (newVal) localStorage.setItem('sbt', 1)
                await this.fetchData()
            }
        }
    },
    components: {
        TopBtns
    }
}
</script>
