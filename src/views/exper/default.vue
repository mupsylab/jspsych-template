<template>
    <div id="exp"></div>
</template>

<script setup lang="ts">
import { initJsPsych } from 'jspsych';
import { onMounted, render, h } from 'vue';

import jsPsychHtmlKeyboardResponse from "@jspsych/plugin-html-keyboard-response";

import { useCheckBrowserInfo } from "../../store/browserCheck";
import { useLoaderAssets } from '../../store/loadAssetsToBlob';
import Session from '../../utils/session';
import { getUuid } from '../../utils/random';
const loader = useLoaderAssets();
const cbi = useCheckBrowserInfo();
const session = new Session();

const jsPsych = initJsPsych({
    display_element: "exp"
});

const timeline: object[] = [{
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<span id='a1'>0</span>/<span id='a2'>1</span>",
    on_load() {
        // 初始化代码放这里
        if(!loader.isInit) {
            // 进行加载，同时避免vue的热更新
            loader.addAssets("./logo.svg");
        }
        loader.startLoad();

        const totalNumDom = document.querySelector("#a2") as HTMLDivElement;
        const countNumDom = document.querySelector("#a1") as HTMLDivElement;

        const i = setInterval(() => {
            if (cbi.isInit && loader.isFinish) {
                clearInterval(i);
                jsPsych.finishTrial({
                    type: "init-function"
                });
            } else {
                const { len, left, loading } = loader.progress;
                totalNumDom.innerText = len.toString();
                countNumDom.innerText = (len - left + loading).toFixed(2);
            }
        }, 100);
    }
}];

import questionnaire from '../../components/questionnaire.vue';
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<div id='box'></div>",
    on_load() {
        render(h(questionnaire, {
            title: "测试题",
            desc: "这只是一个示例，展示问卷如何使用",
            questionList: [
                { type: "radio", name: "check", title: "检测题测试", desc: "测试单选题", choices: ["正确答案", "错误答案"] },
                { type: "text", name: "text", title: "测试填空题", desc: "", placeholder: "输入任意内容" },
                { type: "number", name: "num", title: "测试填数字", desc: "", placeholder: "输入任意数字" }
            ],
            onEndTrial(data) {
                jsPsych.finishTrial(data);
            }
        }), document.querySelector("#box") as Element);
    }
});

import endExp from "./endExp.vue";
timeline.push({
    type: jsPsychHtmlKeyboardResponse,
    choices: ["NO_KEYS"],
    stimulus: "<div id='box'></div>",
    on_load() {
        jsPsych.data.write(cbi.browser);
        session.offlineSave(jsPsych.data.get().csv(), getUuid());
        render(h(endExp), document.querySelector("#box") as Element);
    }
});

onMounted(() => {
    jsPsych.run(timeline);
});
</script>