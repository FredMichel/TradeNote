<script setup>
import { onMounted, onBeforeMount } from 'vue'
import SpinnerLoadingPage from '../components/SpinnerLoadingPage.vue';
import NoData from '../components/NoData.vue';
import { spinnerLoadingPage, diaries, selectedItem, spinnerLoadMore, endOfList, tags, satisfactionArray } from '../stores/globals';
import { useCheckVisibleScreen, useCreatedDateFormat, useEditItem, useInitPopover, useLoadMore } from '../utils/utils';
import { useGetDiaries } from '../utils/diary';
import { useGetTags, useGetTagInfo, useGetAvailableTags, useDailySatisfactionChange, useGetSatisfactions } from '../utils/daily';

onBeforeMount(async () => {

})

onMounted(async () => {
    await useGetDiaries(true)
    await Promise.all([useGetTags(), useGetAvailableTags(), useGetSatisfactions()])
    useInitPopover()
    window.addEventListener('scroll', () => {
        let scrollTop = window.scrollY
        let visibleScreen = window.innerHeight
        let documentHeight = document.documentElement.scrollHeight
        let difference = documentHeight - (scrollTop + visibleScreen)

        if (difference <= 0) {
            if (!spinnerLoadMore.value && !spinnerLoadingPage.value && !endOfList.value) { //To avoid firing multiple times, make sure it's not loadin for the first time and that there is not already a loading more (spinner)
                useLoadMore()
            }
        }
    })
    useCheckVisibleScreen()
})
</script>

<template>
    <SpinnerLoadingPage />
    <div v-show="!spinnerLoadingPage" class="row mt-2 mb-2">
        <div v-if="diaries.length == 0">
            <NoData />
        </div>
        <div class="col-12">
            <div v-for="(itemDiary, index) in diaries" class="row mt-2">
                <div class="col-12">
                    <div class="dailyCard quill">
                        <div class="row">
                            <div class="col-12 cardFirstLine">
                                <div class=" d-flex align-items-center fw-bold">
                                    <div class="col-auto">{{ useCreatedDateFormat(itemDiary.dateUnix) }}

                                        <i v-on:click="useDailySatisfactionChange(itemDiary.dateUnix, true)"
                                            v-bind:class="[(satisfactionArray.find(obj => obj.dateUnix == itemDiary.dateUnix) != undefined && satisfactionArray.find(obj => obj.dateUnix == itemDiary.dateUnix).satisfaction == true) ? 'greenTrade' : '', 'uil', 'uil-thumbs-up', 'ms-2', 'me-1', 'pointerClass']"></i>
                                        <i v-on:click="useDailySatisfactionChange(itemDiary.dateUnix, false)"
                                            v-bind:class="[(satisfactionArray.find(obj => obj.dateUnix == itemDiary.dateUnix) != undefined && satisfactionArray.find(obj => obj.dateUnix == itemDiary.dateUnix).satisfaction == false) ? 'redTrade' : '', , 'uil', 'uil-thumbs-down', 'pointerClass']"></i>
                                    </div>
                                    <span class="col mb-2 ms-auto text-end">
                                        <i class="uil uil-edit-alt pointerClass"
                                            v-on:click="useEditItem(itemDiary.objectId)"></i>

                                        <i v-on:click="selectedItem = itemDiary.objectId"
                                            class="ps-2 uil uil-trash-alt popoverDelete pointerClass"
                                            data-bs-html="true"
                                            data-bs-content="<div>Are you sure?</div><div class='text-center'><a type='button' class='btn btn-red btn-sm popoverYes'>Yes</a><a type='button' class='btn btn-outline-secondary btn-sm ms-2 popoverNo'>No</a></div>"
                                            data-bs-toggle="popover" data-bs-placement="left"></i>
                                    </span>
                                </div>
                                <div>
                                    <span
                                        v-for="tags in tags.filter(obj => obj.tradeId == itemDiary.dateUnix.toString())">
                                        <span v-for="tag in tags.tags.slice(0, 7)" class="tag txt-small"
                                            :style="{ 'background-color': useGetTagInfo(tag).groupColor }">{{
                                                useGetTagInfo(tag).tagName
                                            }}
                                        </span>
                                        <span v-show="tags.tags.length > 7">+{{
                                            tags.tags.length
                                            - 7 }}</span>
                                    </span>
                                </div>
                            </div>
                            <div class="col-12 mt-2">
                                <p v-html="itemDiary.diary"></p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Load more spinner -->
        <div v-if="spinnerLoadMore" class="d-flex justify-content-center mt-3">
            <div class="spinner-border text-blue" role="status"></div>
        </div>
    </div>
</template>