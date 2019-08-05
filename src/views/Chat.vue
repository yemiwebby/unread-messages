<template>
  <div class="booker">
    <nav-bar :name="this.username" :avatar="this.avatar" />
    <div class="chat">
        <div class="container">
          <div class="msg-header">
              <div class="active">
                  <h5>#General <span class="fa fa-badge"><b>{{ unreadCount }}</b></span></h5>
                   <span class="" @click="logoutUser">Logout</span>
              </div>
          </div>

          <div class="chat-page">
              <div class="msg-inbox">
                  <div class="chats" id="chats">
                      <div class="msg-page" id="msg-page">

                        <div
                          v-if="loadingMessages"
                          class="loading-messages-container"
                        >
                          <spinner :size="100"/>
                          <span class="loading-text">
                            Loading Messages
                          </span>
                        </div>
                        <div class="text-center img-fluid empty-chat" v-else-if="!groupMessages.length" >
                          <div class="empty-chat-holder">
                            <img src="../assets/empty-state.svg" class="img-res" alt="empty chat image">
                          </div>

                          <div>
                            <h2> No new message? </h2>
                            <h6 class="empty-chat-sub-title">
                              Send your first message below.
                            </h6>
                          </div>
                        </div>

                        <div v-else>
                          <div v-for="message in groupMessages" v-bind:key="message.id">
                            <div class="received-chats" v-if="message.sender.uid != uid">
                                <div class="received-chats-img">
                                  <img v-bind:src="message.sender.avatar" alt="" class="avatar">
                                </div>

                                <div class="received-msg">
                                    <div class="received-msg-inbox">
                                        <p><span>{{ message.sender.uid }}</span><br>{{ message.data.text }}</p>
                                    </div>
                                </div>
                            </div>


                            <div class="outgoing-chats" v-else>
                                  <div class="outgoing-chats-msg">
                                      <p>{{ message.data.text }}</p>
                                  </div>

                                  <div v-if="sent && messageId == message.id" style="color: green !important;position: relative;">
                                    <!-- <div v-if="sent && messageId == message.id" style="color: green !important;position: relative; top: 40px; right: 30px;"> -->
                                     <div style="position: absolute; right: 0; bottom: 0;padding: 10px 10px 0 0;">
                                       <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px"
width="23" height="23"
viewBox="0 0 48 48"
style=" fill:#000000;"><path fill="#43A047" d="M40.6 12.1L17 35.7 7.4 26.1 4.6 29 17 41.3 43.4 14.9z"></path></svg>
                                    </div>
                                  </div>

                                  <div v-else-if="delivered && messageId == message.id" style="color: green !important; position: relative;">
                                    <div style="position: absolute; right: 0; bottom: 0;padding: 10px 10px 0 0;">
                                      <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px"
width="23" height="23"
viewBox="0 0 226 226"
style=" fill:#000000;"><g fill="none" fill-rule="nonzero" stroke="none" stroke-width="1" stroke-linecap="butt" stroke-linejoin="miter" stroke-miterlimit="10" stroke-dasharray="" stroke-dashoffset="0" font-family="none" font-weight="none" font-size="none" text-anchor="none" style="mix-blend-mode: normal"><path d="M0,226v-226h226v226z" fill="none"></path><g fill="#2ecc71"><path d="M212.68483,42.375l-109.1015,109.90192l-43.17071,-43.98525l-13.32929,13.43758l56.5,57.18742l122.41667,-123.396z"></path><path d="M165.6015,42.375l-109.1015,109.90192l-43.17071,-43.98525l-13.32929,13.43758l56.5,57.18742l122.41667,-123.396z"></path></g></g></svg>
                                      
                                    </div>
                                  </div>

                                  <div v-else-if="read && messageId == message.id" style="color: blue !important; position: relative;">
                                   <div style="position: absolute; right: 0; bottom: 0;padding: 10px 10px 0 0;">
                                     <svg xmlns="http://www.w3.org/2000/svg" x="0px" y="0px" width="23" height="23" viewBox="0 0 226 226" style=" fill:#000000;">
                                        <g fill="none" fill-rule="nonzero" stroke="none"
                                         stroke-width="1" 
                                         stroke-linecap="butt" 
                                         stroke-linejoin="miter" 
                                         stroke-miterlimit="10" 
                                         stroke-dasharray="" 
                                         stroke-dashoffset="0" 
                                         font-family="none" 
                                         font-weight="none" 
                                         font-size="none" 
                                         text-anchor="none" 
                                         style="mix-blend-mode: normal">
                                          <path d="M0,226v-226h226v226z" fill="none"></path>
                                        <g>
                                        <path d="M212.68483,42.375l-109.1015,109.90192l-43.17071,-43.98525l-13.32929,13.43758l56.5,57.18742l122.41667,-123.396z" fill="#0c9eff"></path>
                                        <path d="M165.6015,42.375l-109.1015,109.90192l-43.17071,-43.98525l-13.32929,13.43758l56.5,57.18742l122.41667,-123.396z" fill="#06a3ff"></path>
                                          </g>
                                          </g>
                                      </svg>
                                  </div>

                                  <div class="outgoing-chats-img">
                                      <img v-bind:src="message.sender.avatar" alt="" class="avatar">
                                  </div>
                              </div>
                          </div>
                        </div>
                      </div>
                  </div>
              </div>

              <div class="msg-bottom">
                <form class="message-form" v-on:submit.prevent="sendGroupMessage">
                  <div class="input-group">
                    <input type="text" class="form-control message-input" placeholder="Type something" v-model="chatMessage" required>
                    <spinner
                      v-if="sendingMessage"
                      class="sending-message-spinner"
                      :size="30"
                    />
                  </div>
                </form>
              </div>
          </div>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import { CometChat } from "@cometchat-pro/chat";
import NavBar from "../components/NavBar.vue";
import Spinner from "../components/Spinner.vue";

export default {
  name: "home",
  components: {
    NavBar,
    Spinner
  },
  data() {
    return {
      username: "",
      avatar: "",
      uid: "",
      messageId: "",
      sendingMessage: false,
      chatMessage: "",
      loggingOut: false,
      groupMessages: [],
      loadingMessages: false,
      sent: false,
      delivered: false,
      read: false,
      unreadCount: 0
    };
  },
  mounted() {
    this.loadingMessages = true;
    var listenerID = "UNIQUE_LISTENER_ID";

    const messagesRequest = new CometChat.MessagesRequestBuilder()
      .setLimit(100)
      .build();
    messagesRequest.fetchPrevious().then(
      messages => {
        console.log("Message list fetched:", messages);
        console.log("this.groupMessages", this.groupMessages);
        this.groupMessages = [...this.groupMessages, ...messages];
        this.loadingMessages = false;
        this.$nextTick(() => {
          this.scrollToBottom();
        });
      },
      error => {
        console.log("Message fetching failed with error:", error);
      }
    );

    CometChat.addMessageListener(
      listenerID,
      new CometChat.MessageListener({
        onTextMessageReceived: textMessage => {
          console.log("Text message received successfully", textMessage);
          // Handle text message
          console.log(this.groupMessages);
          this.groupMessages = [...this.groupMessages, textMessage];
          this.loadingMessages = false;

          CometChat.markMessageAsRead(textMessage);
          console.log("This was marked as read", textMessage);

          this.$nextTick(() => {
            this.scrollToBottom();
          });
        },
        onMessageDelivered: messageReceipt => {
          this.messageId = messageReceipt.messageId;
          this.read = false;
          this.delivered = true;
          this.sent = false;
          console.log("MessageDelivered", { messageReceipt });
        },
        onMessageRead: messageReceipt => {
          console.log("MessageRead", { messageReceipt });

          this.read = true;
          this.delivered = false;
          this.sent = false;
        }
      })
    );
  },

  created() {
    this.getLoggedInUser();
    let GUID = "supergroup";
    CometChat.getUnreadMessageCountForGroup(GUID).then(
      data => {
        console.log("Message count fetched for unread", data);
        console.log(data);
        this.unreadCount = data.supergroup;
      },
      error => {
        console.log("Error in getting message count", error);
      }
    );
  },
  methods: {
    getLoggedInUser() {
      CometChat.getLoggedinUser().then(
        user => {
          this.username = user.name;
          this.avatar = user.avatar;
          this.uid = user.uid;
        },
        error => {
          this.$router.push({
            name: "homepage"
          });
          console.log(error);
        }
      );
    },

    scrollToBottom() {
      const chat = document.getElementById("msg-page");
      chat.scrollTo(0, chat.scrollHeight + 30);
    },

    sendGroupMessage() {
      this.sendingMessage = true;
      var receiverID = process.env.VUE_APP_COMMETCHAT_GUID;
      var messageText = this.chatMessage;
      var messageType = CometChat.MESSAGE_TYPE.TEXT;
      var receiverType = CometChat.RECEIVER_TYPE.GROUP;
      let globalContext = this;

      var textMessage = new CometChat.TextMessage(
        receiverID,
        messageText,
        messageType,
        receiverType
      );

      CometChat.sendMessage(textMessage).then(
        message => {
          console.log("Message sent successfully:", message);
          this.chatMessage = "";
          this.sendingMessage = false;
          // Text Message Sent Successfully
          this.groupMessages = [...globalContext.groupMessages, message];

          this.messageId = message.id;

          this.read = false;
          this.delivered = false;
          this.sent = true;

          this.$nextTick(() => {
            this.scrollToBottom();
          });
        },
        error => {
          console.log("Message sending failed with error:", error);
        }
      );
    },
    logoutUser() {
      CometChat.logout().then(
        success => {
          console.log("Logout completed successfully");
          this.$router.push({ name: "homepage" });
          console.log(success);
        },
        error => {
          console.log("Logout failed with exception:", { error });
        }
      );
    }
  }
};
</script>
