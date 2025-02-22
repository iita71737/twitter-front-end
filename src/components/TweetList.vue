<template>
  <div class="tweet-list" v-if="tweets">
    <div
      v-for="tweet in tweets"
      :key="`${tweet.id}-${Math.random()}`"
      class="list-item"
    >
      <div
        class="avatar"
        :style="{
          background: `url(${tweet.avatar}) no-repeat center/cover`,
        }"
        @click="$router.push(`/users/${tweet.UserId}`).catch(() => {})"
      ></div>
      <div class="tweet-wrapper">
        <div class="info">
          <div
            class="name"
            @click="$router.push(`/users/${tweet.UserId}`).catch(() => {})"
          >
            {{ tweet.name }}
          </div>
          <div class="account-and-post-time">
            <span
              class="account"
              @click="$router.push(`/users/${tweet.UserId}`).catch(() => {})"
              >＠{{ tweet.account }} </span
            >&bull;
            <span>{{ tweet.createdAt | fromNow }}</span>
          </div>
        </div>
        <div
          @click="$router.push(`/users/${tweet.Tweet.UserId}`).catch(() => {})"
          class="reply-to-wrapper"
          v-show="tweet.type === 'reply'"
        >
          <span class="title">回覆 </span>
          <span class="name">@{{ tweet.replyTo }}</span>
        </div>
        <div class="content" @click="tweetDetail(tweet)">
          {{ tweet.description }}
        </div>
        <div class="action" v-show="tweet.type !== 'reply'">
          <div class="reply-wrapper">
            <div class="icon reply" @click="afterClickNewReply($event)"></div>
            <span class="number">{{ tweet.replyTweetCount }}</span>
          </div>
          <div class="like-wrapper">
            <div
              v-if="!tweet.isLiked"
              class="icon like"
              @click="likeTweet(tweet.id)"
            ></div>
            <div
              v-if="tweet.isLiked"
              class="icon like liked"
              @click="unlikeTweet(tweet.id)"
            ></div>
            <span class="number">{{ tweet.likeTweetCount }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { fromNowFilter } from "../utils/mixins.js";
//import tweetsAPI from "@/apis/tweets";
import { Toast } from "@/utils/helpers";
import likesAPI from "@/apis/likes";

export default {
  name: "TweetList",
  components: {},
  mixins: [fromNowFilter],
  props: {
    tweets: {
      type: Array,
      default: () => {
        return [];
      },
    },
    isReply: Boolean,
  },
  data() {
    return {};
  },
  methods: {
    tweetDetail(tweet) {
      if (this.isReply) {
        this.$router.push(`/reply_list/${tweet.id}`);
      }
    },
    async likeTweet(tweetId) {
      try {
        const { data } = await likesAPI.likeTweet({ tweetId });
        console.log("likeTweet", data);
        if (data.status !== "success") {
          throw new Error(data.message);
        }

        this.tweets.filter((tweet) => {
          if (tweet.id === tweetId) {
            tweet.isLiked = true;
            tweet.likeTweetCount =
              tweet.likeTweetCount + 1 > 0 ? tweet.likeTweetCount + 1 : 0;
          }
        });
        this.$emit("update-from-child", tweetId);
      } catch (error) {
        console.log(error);
        Toast.fire({
          icon: "error",
          title: "無法按讚推文，請稍後再試",
        });
      }
    },
    async unlikeTweet(tweetId) {
      try {
        const { data } = await likesAPI.unlikeTweet({ tweetId });
        console.log("unlikeTweet", data);
        if (data.status !== "success") {
          throw new Error(data.message);
        }

        this.tweets.filter((tweet) => {
          if (tweet.id === tweetId) {
            tweet.isLiked = false;
            tweet.likeTweetCount =
              tweet.likeTweetCount - 1 <= 0 ? 0 : tweet.likeTweetCount - 1;
          }
        });
        this.$emit("update-from-child", tweetId);
      } catch (error) {
        console.log(error);
        Toast.fire({
          icon: "error",
          title: "無法按讚推文，請稍後再試",
        });
      }
    },
  },
};
</script>

<style lang="scss" scoped>
$orange: #ff6600;
$deeporange: #f34a16;
$lightgray: #f6f8fa;
$dark: #2c3e50;
$bitdark: #657786;
$divider: #e6ecf0;
.tweet-list {
  width: 100%;
  .list-item {
    padding: 10px 0 10px 0;
    height: 146px;
    border-bottom: 1px solid $divider;
    display: flex;
    flex-direction: row;
    position: relative;
    cursor: pointer;
    transition: ease-in 0.2s;
    &:hover {
      backdrop-filter: brightness(0.95);
    }
    .avatar {
      margin: 3px 0 0 15px;
      height: 50px;
      min-width: 50px;
      border-radius: 100px;
      background: url(https://source.unsplash.com/collection/4389261/100x100)
        no-repeat center;
      background-size: 100%;
      cursor: pointer;
      &:hover {
        filter: brightness(0.9);
      }
    }
    .tweet-wrapper {
      margin-left: 10px;
      .info {
        height: 22px;
        line-height: 22px;
        display: flex;
        flex-direction: row;
        font-size: 15px;
        font-weight: 700;
        .name {
          font-weight: 700;
          margin-right: 5px;
          cursor: pointer;
          &:hover {
            text-decoration: underline;
          }
        }
        .account-and-post-time {
          color: $bitdark;
          cursor: pointer;
          .account {
            &:hover {
              text-decoration: underline;
            }
          }
        }
      }
      .reply-to-wrapper {
        margin-top: 4px;
        text-align: left;
        height: 22px;
        line-height: 22px;
        font-size: 15px;
        font-weight: 500;
        cursor: pointer;
        &:hover {
          text-decoration: underline;
        }
        .title {
          color: $bitdark;
        }
        .name {
          color: $orange;
        }
      }
      .content {
        width: 100%;
        max-width: 510px;
        height: 66px;
        overflow: hidden;
        margin: 5px 15px 0 0;
        font-size: 15px;
        font-weight: 500;
        line-height: 22px;
        text-align: left;
      }
      .action {
        width: 130px;
        height: 21px;
        margin-top: 10px;
        display: flex;
        flex-direction: row;
        justify-content: space-between;
        color: $bitdark;
        z-index: 999;
        .icon {
          max-width: 15px;
          width: 100%;
          height: 100%;
          background-color: $bitdark;
        }
        .icon.reply {
          mask: url(../assets/svgs/icon_reply.svg) no-repeat center;
          mask-size: contain;
        }
        .icon.like {
          mask: url(../assets/svgs/icon_like.svg) no-repeat center;
          mask-size: contain;
        }
        .icon.like.liked {
          mask: url(../assets/svgs/icon_liked.svg) no-repeat center;
          mask-size: contain;
          background-color: #e0245e;
        }
        .reply-wrapper {
          display: flex;
          align-items: center;
          width: 40px;
          margin: 3px 0;
          height: 15px;
          cursor: pointer;
          transition: ease-in 0.2s;
          &:hover {
            .icon {
              background-color: $orange;
            }
          }
        }
        .like-wrapper {
          display: flex;
          align-items: center;
          width: 40px;
          margin: 3px 0;
          height: 15px;
          cursor: pointer;
          transition: ease-in 0.2s;
          &:hover {
            .icon.like {
              mask: url(../assets/svgs/icon_liked.svg) no-repeat center;
              mask-size: contain;
              background-color: $orange;
            }
          }
        }
        .number {
          width: 15px;
          height: 13px;
          margin-left: 10px;
          font-weight: 500;
          font-size: 13px;
          line-height: 13px;
        }
      }
    }
  }
}
</style>
