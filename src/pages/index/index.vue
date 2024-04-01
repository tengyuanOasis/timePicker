<template>
	<view class="content">
		<button
			type="primary"
			@click="updateVisible(true)"
		>
			打开弹窗
		</button>
		<MyTimePicker
			:visible="visible"
			:showFooter="showFooter"
			:autoClose="autoClose"
			:selectedDateProp="selectedDateProp"
			:selectedTimeProp="selectedTimeProp"
			:timeConfig="timeConfig"
			@selectTime="selectTime"
			@handleSubmit="handleSubmit"
			@handleCancel="handleCancel"
		/>
	</view>
</template>

<script setup>
import { ref } from 'vue';
import MyTimePicker from '../../components/TimePicker.vue';
const visible = ref(false);
const showFooter = ref(true);
const autoClose = ref(true);
const selectedDateProp = ref({});
const selectedTimeProp = ref('');
const timeConfig = ref({
	showRightNow: true,
	startTime: '9:00',
	endTime: '20:00',
	difference: 60,
});

const updateVisible = (bol) => {
	visible.value = bol;
	console.log('%c [ visible.value ]-39', 'font-size:13px; background:#0094fb; color:#fff;', visible.value);
};
const selectTime = (date, time) => {
	console.log(date, time);
};

const handleSubmit = (date, time) => {
	console.log('%c [ time ]-47', 'font-size:13px; background:#0094fb; color:#fff;', time);
	console.log('%c [ date ]-47', 'font-size:13px; background:#0094fb; color:#fff;', date);
	visible.value = false;
};

const handleCancel = () => {
	visible.value = false;
};
</script>

<style>
.content {
	width: 100%;
	height: 100%;
}
</style>
