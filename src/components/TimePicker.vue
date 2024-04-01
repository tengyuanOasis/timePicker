<template>
	<uni-popup
		ref="timePicker"
		type="bottom"
		:maskClick="false"
		@close="handleClose"
	>
		<div class="date_pop">
			<div class="popup-header">
				<div class="pop-title">请选择取件时间</div>
			</div>

			<div class="popup-content">
				<!-- 日期 -->
				<div class="date_box">
					<div
						v-for="(date, index) in effectRecentDate"
						:key="date.date_zh"
						:class="[`date_item`, selectedDate.date_zh == date.date_zh ? `date_active` : ``]"
						@click="timeSelect('date', index, date)"
					>
						{{ date.date_zh }}({{ date.week }})
					</div>
				</div>

				<!-- 时间 -->
				<scroll-view
					class="time_box"
					scroll-y
					scroll-with-animation
					:scroll-into-view="scrollIntoViewId"
				>
					<div
						v-for="(time, index) in effectAppointmentTime"
						:id="`time_${index}`"
						:key="index"
						:class="{
							time_item: true,
							time_active: selectedTime === time,
						}"
						@click="timeSelect('time', index)"
					>
						{{ time }}
					</div>
				</scroll-view>
			</div>

			<div
				class="popup-footer"
				v-if="showFooter"
			>
				<div
					class="footer-btn close"
					@click="handleClose"
				>
					取消
				</div>
				<div
					class="footer-btn submit"
					@click="submit"
				>
					提交
				</div>
			</div>
		</div>
	</uni-popup>
</template>

<script setup>
import { ref, computed, watch, onMounted, defineProps, defineEmits } from 'vue';
import dayjs from 'dayjs';

const props = defineProps({
	visible: { type: Boolean, required: true, default: false },
	showFooter: { type: Boolean, required: false, default: false },
	autoClose: { type: Boolean, required: false, default: true },
	selectedDateProp: { type: Object, required: true, default: () => ({}) },
	selectedTimeProp: { type: String, required: true, default: '' },
	timeConfig: {
		type: Object,
		required: true,
		default: () => ({
			startTime: '9:00',
			endTime: '20:00',
			difference: 60,
		}),
	},
});

const emit = defineEmits(['selectTime', 'handleSubmit', 'handleCancel']);

const currentDate = dayjs().format('YYYY/MM/DD');

const selectedTimeIdx = ref(0);

const defaultTimeList = [
	'08:00-09:00',
	'09:00-10:00',
	'10:00-11:00',
	'11:00-12:00',
	'12:00-13:00',
	'13:00-14:00',
	'14:00-15:00',
	'15:00-16:00',
	'16:00-17:00',
	'17:00-18:00',
	'18:00-19:00',
	'19:00-20:00',
];

const timePickerConfig = ref({
	dayCount: 3,
	timeListStartHour: 9,
	timeListEndHour: 20,
	delayMinutes: 60,
	showRightNow: true,
});

const recentDateList = ref([]);
const effectRecentDate = ref([]);
const effectAppointmentTime = ref([]);
const appointment = ref([]);
const selectedDate = ref({});
const selectedTime = ref('');
const timePicker = ref(null);

const show = () => {
	timePicker.value.open();
};

const close = () => {
	timePicker.value.close();
};

const setSelectDate = () => {
	setEffectAppointmentTime();
	setEffectRecentDate();
	selectedTime.value = effectAppointmentTime.value[0];
	selectedDate.value = effectRecentDate.value[0];
};

const submit = () => {
	emit('handleSubmit', selectedDate.value, selectedTime.value);
};

const handleClose = () => {
	emit('handleCancel', selectedDate.value, selectedTime.value);
};

watch(
	() => props.visible,
	(newVal) => {
		if (newVal) {
			setEffectAppointmentTime();
			setEffectRecentDate();
			show();
		} else {
			close();
		}
	},
);

onMounted(() => {
	setRecentData();
	loadReservationTime();
	setTimeout(() => {
		setSelectDate();
	}, 1000);
});

const loadReservationTime = () => {
	const conf = props.timeConfig;
	const newConfig = {
		...timePickerConfig.value,
		timeListStartHour: Number(conf?.startTime?.replace(':00', '')),
		timeListEndHour: Number(conf?.endTime?.replace(':00', '')),
		delayMinutes: conf?.difference,
		showRightNow: conf?.showRightNow ?? false,
	};
	timePickerConfig.value = newConfig;
	createTimeList();
};

const createTimeList = () => {
	const { timeListStartHour, timeListEndHour } = timePickerConfig.value;
	const createList = (item) => {
		let hour = Number(item.slice(0, 2));
		if (hour >= timeListStartHour && hour < timeListEndHour) {
			return item;
		}
	};
	const setEffectList = (val) => val;
	const timeList = defaultTimeList.map(createList).filter(setEffectList);
	effectAppointmentTime.value = timeList;
	appointment.value = timeList;
};

const timeSelect = (type, index, date) => {
	if (type === 'time') {
		selectedTimeIdx.value = index;
		timeChange(effectAppointmentTime.value[index], 'time');
	} else {
		timeChange(date, 'date');
		selectedTimeIdx.value = 0;
	}
};

const timeChange = (date, type) => {
	const dateList = recentDateList.value;
	if (type === 'date') {
		selectedDate.value = date;
		selectedTime.value = '';
		setEffectAppointmentTime();
	} else {
		if (selectedDate.value.date_zh == '') {
			selectedDate.value = dateList[0];
		}
		selectedTime.value = date;
		if (props.autoClose) {
			handleClose();
		}
	}
	emit('update:selectedDateProp', selectedDate.value);
	emit('update:selectedTimeProp', selectedTime.value);
	emit('selectTime', selectedDate.value, selectedTime.value);
};

const setEffectRecentDate = () => {
	if (effectAppointmentTime.value.length > 0) {
		effectRecentDate.value = JSON.parse(JSON.stringify(recentDateList.value));
		return;
	}

	let list = recentDateList.value;

	list.splice(0, 1);
	effectRecentDate.value = list;

	selectedDate.value = recentDateList.value[0];

	setEffectAppointmentTime();
};

const setEffectAppointmentTime = () => {
	const appointmentList = appointment.value;

	if (selectedDate.value.date_en != currentDate) {
		effectAppointmentTime.value = appointmentList;
		return;
	}

	let list = appointmentList.filter((item) => checkpxainingMinute(item) === 3);

	const { timeListStartHour, timeListEndHour, showRightNow = true } = timePickerConfig.value;

	if (new Date().getHours() >= timeListStartHour && new Date().getHours() < timeListEndHour - 1 && showRightNow) {
		list.unshift('立即上门');
	}

	effectAppointmentTime.value = list;
};

const setRecentData = (n = timePickerConfig.value.dayCount) => {
	const oneDayTime = 60 * 1000 * 60 * 24;
	const today = +new Date();
	let list = [];
	for (let i = 0; i < n; i++) {
		let formatTime = formatTime_zh(today + oneDayTime * i);
		list.push({
			...formatTime,
			week: i == 0 ? '今天' : i == 1 ? '明天' : i == 2 ? '后天' : formatTime.week,
		});
	}
	selectedDate.value = list[0];
	recentDateList.value = list;
};

const formatTime_zh = (date) => {
	date = new Date(date);
	const year = date.getFullYear();
	const month = date.getMonth() + 1;
	const day = date.getDate();
	const weekDay = date.getDay();
	const formatNumber = (n) => {
		n = n.toString();
		return n[1] ? n : '0' + n;
	};
	const numToTxt = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
	return {
		date_zh: `${formatNumber(month)}月${formatNumber(day)}日`,
		date_en: `${year}/${formatNumber(month)}/${formatNumber(day)}`,
		week: numToTxt[weekDay],
	};
};

const checkpxainingMinute = (time) => {
	const delayMinutes = timePickerConfig.value?.delayMinutes ?? 60;
	if (!time) return;
	const outTime = time.toString().split('-')[0];
	const now = dayjs().valueOf();
	const dateTime = currentDate + ' ' + outTime;
	const check = dayjs(dateTime).valueOf();
	const difference = check - now;
	const minutes = difference / (1000 * 60);
	return minutes <= 0 ? 1 : minutes <= delayMinutes ? 2 : 3;
};

const scrollIntoViewId = computed(() => `time_${selectedTimeIdx.value}`);

defineExpose({
	show,
	close,
});
</script>

<style scoped lang="scss">
@import './TimePicker.scss';
</style>
