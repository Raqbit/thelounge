<template>
	<div id="chat-container" class="window" :data-current-channel="channel.name">
		<div
			id="chat"
			:data-id="channel.id"
			:class="{
				'hide-motd': !this.$root.settings.motd,
				'colored-nicks': this.$root.settings.coloredNicks,
				'show-seconds': this.$root.settings.showSeconds,
			}"
		>
			<div
				:id="'chan-' + channel.id"
				:class="[channel.type, 'chan', 'active']"
				:data-id="channel.id"
				:data-type="channel.type"
				:aria-label="channel.name"
				role="tabpanel"
			>
				<div class="header">
					<button class="lt" aria-label="Toggle channel list" />
					<span class="title">{{ channel.name }}</span>
					<div v-if="channel.editTopic === true" class="topic-container">
						<input
							:value="channel.topic"
							class="topic-input"
							placeholder="Set channel topic"
							@keyup.enter="saveTopic"
							@keyup.esc="channel.editTopic = false"
						/>
						<span aria-label="Save topic" class="save-topic" @click="saveTopic">
							<span type="button" aria-label="Save topic"></span>
						</span>
					</div>
					<span v-else :title="channel.topic" class="topic" @dblclick="editTopic"
						><ParsedMessage
							v-if="channel.topic"
							:network="network"
							:text="channel.topic"
					/></span>
					<button class="menu" aria-label="Open the context menu" />
					<span
						v-if="channel.type === 'channel'"
						class="rt-tooltip tooltipped tooltipped-w"
						aria-label="Toggle user list"
					>
						<button class="rt" aria-label="Toggle user list" />
					</span>
				</div>
				<div v-if="channel.type === 'special'" class="chat-content">
					<div class="chat">
						<div class="messages">
							<div class="msg">
								<Component
									:is="specialComponent"
									:network="network"
									:channel="channel"
								/>
							</div>
						</div>
					</div>
				</div>
				<div v-else class="chat-content">
					<div
						:class="[
							'scroll-down tooltipped tooltipped-w tooltipped-no-touch',
							{'scroll-down-shown': !channel.scrolledToBottom},
						]"
						aria-label="Jump to recent messages"
						@click="$refs.messageList.jumpToBottom()"
					>
						<div class="scroll-down-arrow" />
					</div>
					<MessageList ref="messageList" :network="network" :channel="channel" />
					<ChatUserList v-if="channel.type === 'channel'" :channel="channel" />
				</div>
			</div>
		</div>
		<div
			v-if="this.$root.currentUserVisibleError"
			id="user-visible-error"
			@click="hideUserVisibleError"
		>
			{{ this.$root.currentUserVisibleError }}
		</div>
		<span id="upload-progressbar" />
		<ChatInput :network="network" :channel="channel" />
	</div>
</template>

<script>
const socket = require("../js/socket");
import ParsedMessage from "./ParsedMessage.vue";
import MessageList from "./MessageList.vue";
import ChatInput from "./ChatInput.vue";
import ChatUserList from "./ChatUserList.vue";
import ListBans from "./Special/ListBans.vue";
import ListInvites from "./Special/ListInvites.vue";
import ListChannels from "./Special/ListChannels.vue";
import ListIgnored from "./Special/ListIgnored.vue";

export default {
	name: "Chat",
	components: {
		ParsedMessage,
		MessageList,
		ChatInput,
		ChatUserList,
	},
	props: {
		network: Object,
		channel: Object,
	},
	computed: {
		specialComponent() {
			switch (this.channel.special) {
				case "list_bans":
					return ListBans;
				case "list_invites":
					return ListInvites;
				case "list_channels":
					return ListChannels;
				case "list_ignored":
					return ListIgnored;
			}

			return undefined;
		},
	},
	methods: {
		hideUserVisibleError() {
			this.$root.currentUserVisibleError = null;
		},
		editTopic() {
			if (this.channel.type === "channel") {
				this.channel.editTopic = true;

				this.$nextTick(() => {
					document.querySelector(`#chan-${this.channel.id} .topic-input`).focus();
				});
			}
		},
		saveTopic() {
			this.channel.editTopic = false;
			const newTopic = document.querySelector(`#chan-${this.channel.id} .topic-input`).value;

			if (this.channel.topic !== newTopic) {
				const target = this.channel.id;
				const text = `/raw TOPIC ${this.channel.name} :${newTopic}`;
				socket.emit("input", {target, text});
			}
		},
	},
};
</script>
