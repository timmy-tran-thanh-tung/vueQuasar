<template>
  <q-layout view="hHh lpR fFf">
    <!-- Drawer (Left Panel) -->
    <q-drawer v-model="leftDrawerOpen" side="left" elevated>
      <q-list>
        <q-item-label header>📜 Lịch sử các cuộc hội thoại</q-item-label>
        <q-item :class="{ 'active-chat': chat.id === currentChatId }" v-for="(chat, index) in chatHistory" :key="index"
          clickable @click="loadChat(index)" @mouseover="hoverIndex = index" @mouseleave="hoverIndex = null">
          <q-item-section>
            <q-item-label>{{ chat.title }}</q-item-label>
          </q-item-section>
          <q-item-section side>
            <q-btn v-show="hoverIndex === index" flat round dense icon="delete" color="#FFFFFF"
              @click.stop="removeChat(index)" />
          </q-item-section>
        </q-item>


      </q-list>
    </q-drawer>

    <q-page-container>
      <q-page class="full-height flex flex-center">
        <q-card class="chat-container">
          <q-card-section class="title row justify-between items-center">
            <div class="row q-gutter-sm">
              <q-btn flat dense round icon="menu" @click="toggleLeftDrawer" />
              <q-btn label="New Chat" color="primary" icon="chat" @click="newChat" />
              <!-- Checkbox -->
              <q-checkbox v-model="checked" label="" color="primary" :true-value="true" :false-value="false" />
              <!-- Hiển thị thông báo khi checkbox được chọn -->
              <p class="p-box active" v-if="checked">Status: Show Think</p>
              <p class="p-box" v-else>Status: Not Show Think</p>
            </div>
            <span>💬 Deepseek nhưng ở một diễn biến khác</span>
          </q-card-section>

          <q-card-section class="full-height chat-box">
            <q-scroll-area ref="chatBox" style="height: 300px; padding-right: 1.2%;">
              <div v-for="(message, index) in messages" :key="index" class="row no-wrap q-mb-sm"
                :class="message.role === 'user' ? 'justify-end' : 'justify-start'">
                <p class="shadow-2 q-pa-md text-white"
                  style="margin: 6px; padding: 6px; padding-inline: 12px; border-radius: 8px; width: fit-content; max-width: 75%;"
                  :class="message.role === 'user' ? 'bg-primary text-right' : 'bg-secondary text-left'">
                  {{ message.role === 'user' ? 'Bạn: ' : '💬 Trả lời: ' }}
                  <span v-html="convertToHtml(message.content)"></span>
                </p>
              </div>
              <!-- Hiển thị dấu ba chấm khi đang chờ phản hồi -->
              <div v-if="isLoading" class="loading-container">
                <q-spinner-dots color="primary" size="50px" />
                <div class="loading-text">
                  <q-badge label="Đang trả lời..." color="grey-5" text-color="white" />
                </div>
              </div>
            </q-scroll-area>
          </q-card-section>

          <q-card-section class="input-area row items-center q-gutter-sm">
            <q-input v-model="userInput" placeholder="Nhập tin nhắn..." @keyup.enter="sendMessage" class="col-9" />
            <q-btn label="Gửi" @click="sendMessage" color="primary" class="col-3" :disabled="isLoading" />
          </q-card-section>
        </q-card>
      </q-page>
    </q-page-container>
  </q-layout>
</template>


<script>
export default {
  data() {
    return {
      checked: false,
      isLoading: false, // Thêm trạng thái loading
      hoverIndex: null,
      leftDrawerOpen: false,
      userInput: "",
      messages: [
        { role: "system", content: "Tôi là Deepseek, chỉ nói tiếng Việt rõ ràng và dễ hiểu." }
      ],
      chatHistory: [],
      currentChatId: null, // ID của cuộc trò chuyện đang mở
      API_CONFIG: {
        url: "https://api.groq.com/openai/v1/chat/completions",
        key: "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",
        model: "deepseek-r1-distill-qwen-32b",
        temperature: 0.8,
        max_tokens: 1024
      },
    };
  },

  methods: {
    // Hàm ngăn tấn công
    escapeHtml(text) {
      return text.replace(/[&<>"']/g, (match) => {
        const escapeChars = { "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#039;" };
        return escapeChars[match];
      });
    },

    // Hàm chuyển đổi nội dung
    convertToHtml(content) {
      console.log('<pre>' + content + '</pre>');
      if (!content) return "";

      // Xử lý danh sách số thứ tự (1., 2., 3.)
      content = content.replace(/(?:^|\n)(\d+)\.\s+(.*)/g, "<li>$2</li>");
      content = content.replace(/(<li>.*<\/li>)+/g, "<ol>$&</ol>");

      // Xử lý dấu \[...\] thành MathJax (công thức toán)
      content = content.replace(/\\\[([\s\S]+?)\\\]/g, (match, p1) => `$$${p1}$$`);

      // Xử lý highlight code block ```language\ncode```
      content = content.replace(/```(\w+)?\n([\s\S]+?)```/g, (match, lang, code) => {
        return `<pre style="width: 100%; background-color: black; padding: .8rem; margin: 0"><code style="width: 100%">${this.escapeHtml(code)}</code></pre>`;
      });

      // Xử lý danh sách không thứ tự (- item, * item)
      content = content.replace(/(?:^|\n)[-*]\s+(.*)/g, "<li>$1</li>");
      content = content.replace(/(<li>.*<\/li>)+/g, "<ul>$&</ul>");

      // Xử lý tiêu đề Markdown (#, ##, ###)
      content = content.replace(/(?:^|\n)######\s?(.*)/g, "<h6 style='font-size: 1rem; font-weight: 600'>$1</h6>");
      content = content.replace(/(?:^|\n)#####\s?(.*)/g, "<h5 style='font-size: 1rem; font-weight: 700'>$1</h5>");
      content = content.replace(/(?:^|\n)####\s?(.*)/g, "<h4 style='font-size: 1.1rem; font-weight: 700'>$1</h4>");
      content = content.replace(/(?:^|\n)###\s?(.*)/g, "<h3 style='font-size: 1.2rem; font-weight: 700'>$1</h3>");
      content = content.replace(/(?:^|\n)##\s?(.*)/g, "<h2 style='font-size: 1.3rem; font-weight: 700'>$1</h2>");
      content = content.replace(/(?:^|\n)#\s?(.*)/g, "<h1 style='font-size: 1.4rem; font-weight: 700'>$1</h1>");

      // Xử lý trích dẫn (blockquote `> text`)
      content = content.replace(/(?:^|\n)> (.*)/g, "<blockquote>$1</blockquote>");

      // Xử lý in đậm * *text** và in nghiêng *text*
      content = content.replace(/\*\*(.*?)\*\*/g, "<b>$1</b>");
      content = content.replace(/\*(.*?)\*/g, "<i>$1</i>");

      // Xử lý link Markdown `[title](URL)`
      content = content.replace(/\[([^\]]+)\]\((https?:\/\/[^]+)\)/g, '<a href="$2" target="_blank">$1</a>');

      // Xử lý bảng Markdown | A | B | C |
      content = content.replace(/(?:^|\n)(\|.*?\|)(?:\n|$)/g, '<table><tr><td>$1</td></tr></table>');
      content = content.replace(/\|/g, '</td><td>');

      // Xử lý xuống dòng thành <br>, nhưng không áp dụng trong danh sách hoặc code block
      content = content.replace(/(?<!<\/?(ul|ol|li|pre|code|table|h1|h2|h3|h4|h5|h6|blockquote)[^>]*>)\n/g, "<br>");

      return content;
    },

    // Lưu dữ liệu vào Local Storage
    saveToLocalStorage() {
      localStorage.setItem("chatHistory", JSON.stringify(this.chatHistory));
      localStorage.setItem("currentChatId", this.currentChatId); // Lưu ID hiện tại
    },

    // Tải lại lịch sử khi mở trang
    loadFromLocalStorage() {
      let savedChats = localStorage.getItem("chatHistory");
      if (savedChats) {
        this.chatHistory = JSON.parse(savedChats);
        // Nếu có lịch sử, load cuộc trò chuyện gần nhất
        if (this.chatHistory.length > 0) {
          this.messages = [...this.chatHistory[0].messages];
        }
      }
    },

    // Đưa đẩy thanh lịch sử trò chuyện
    toggleLeftDrawer() {
      this.leftDrawerOpen = !this.leftDrawerOpen;
    },

    // Vẽ tin nhắn lên giao diện và xử lý checkbox Show Think
    async sendMessage() {
      // Ngăn spam nút gửi
      if (!this.isLoading) {
        if (!this.userInput.trim()) return;

        this.messages.push({ role: "user", content: this.userInput });
        this.userInput = "";
        this.$nextTick(this.scrollToBottom);

        // Kích hoạt trạng thái loading
        this.isLoading = true;

        // Gửi yêu cầu API
        let botResponse = await this.fetchGroqResponse(this.messages);

        // Kích hoạt trạng thái loading
        this.isLoading = false; // Tắt trạng thái loading sau khi nhận phản hồi

        if (botResponse && botResponse.choices) {
          let content = botResponse.choices[0].message.content;

          // Checkbox hiện/ẩn thẻ think 
          if (!this.checked) {
            content = content.replace(/<think>.*?<\/think>/gs, "").trim();
          } else {
            content = content.trim();
          }

          this.messages.push({ role: "system", content });
          this.$nextTick(this.scrollToBottom);
        } else {
          this.messages.push({ role: "system", content: "Hệ thống đang quá tải. Vui lòng thử lại sau." });
        }
      }
    },

    // Lưu toàn bộ hội thoại
    saveChat() {
      if (this.messages.length > 1) {
        let lastUserMessage = this.messages[this.messages.length - 1].content;
        let title = lastUserMessage.length > 30 ? lastUserMessage.substring(0, 30) + "..." : lastUserMessage;

        if (!this.currentChatId) {
          this.currentChatId = Date.now(); // Nếu chưa có ID, tạo ID mới
        }

        // Kiểm tra xem ID này đã có trong lịch sử chưa
        let existingChat = this.chatHistory.find(chat => chat.id === this.currentChatId);

        if (existingChat) {
          // Nếu ID đã tồn tại, cập nhật nội dung
          existingChat.messages = [...this.messages];
        } else {
          // Nếu chưa có, thêm mới vào danh sách
          this.chatHistory.unshift({ id: this.currentChatId, title, messages: [...this.messages] });
        }

        this.saveToLocalStorage(); // Lưu vào Local Storage
      }
    },

    // Tạo mới hội thoại
    newChat() {
      this.saveChat(); // Lưu cuộc trò chuyện hiện tại

      // Kiểm tra nếu đã có ID hiện tại thì giữ nguyên, không tạo mới
      if (!this.currentChatId || this.messages.length > 1) {
        this.currentChatId = Date.now();
      }

      this.messages = [{ role: "system", content: "Tôi là Deepseek R1 nói tiếng Việt rõ ràng và dễ hiểu." }];
    },

    // Tải dữ liệu khi truy cập hội thoại cũ
    loadChat(index) {
      let selectedChat = this.chatHistory[index];

      // Xóa cuộc trò chuyện khỏi vị trí cũ
      this.chatHistory.splice(index, 1);

      // Đưa cuộc trò chuyện lên đầu danh sách
      this.chatHistory.unshift(selectedChat);

      this.messages = [...selectedChat.messages];
      this.currentChatId = selectedChat.id; // Dùng lại ID của cuộc trò chuyện cũ

      this.$nextTick(this.scrollToBottom); // Cuộn xuống khi mở cuộc trò chuyện cũ
      this.saveToLocalStorage(); // Lưu lại trạng thái mới
    },

    // Xóa dữ liệu khi bấm thùng rác
    removeChat(index) {
      const deletedChatId = this.chatHistory[index].id;

      // Hiển thị pop-up xác nhận trước khi xoá
      if (confirm("Bạn có chắc chắn muốn xoá cuộc trò chuyện này?")) {
        // Nếu chat bị xóa trùng với chat đang hiển thị, tạo chat mới
        if (deletedChatId === this.currentChatId) {
          this.newChat();
          // Xóa cuộc trò chuyện khỏi danh sách
          this.chatHistory.splice(index, 1);
        } else {
          // Xóa cuộc trò chuyện khỏi danh sách
          this.chatHistory.splice(index, 1);
        }
        this.saveToLocalStorage(); // Lưu lại danh sách sau khi xóa

        // Nếu không còn cuộc trò chuyện nào, tạo một cuộc trò chuyện mới
        if (this.chatHistory.length === 0) {
          this.newChat();
        }
      }
    },

    // Tự cuộn xuống tin nhắn mới nhất
    scrollToBottom() {
      if (this.$refs.chatBox?.setScrollPosition) {
        this.$refs.chatBox.setScrollPosition("vertical", 999999, 300);
      }
    },

    // Fetch lấy dữ liệu từ API Groq
    async fetchGroqResponse(messages) {
      try {
        const response = await fetch(this.API_CONFIG.url, {
          method: "POST",
          headers: {
            "Authorization": `Bearer ${this.API_CONFIG.key}`,
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            model: this.API_CONFIG.model,
            messages: messages,
            temperature: this.API_CONFIG.temperature,
            max_tokens: this.API_CONFIG.max_tokens
          })
        });
        if (!response.ok) throw new Error(`HTTP Error: ${response.status}`);
        return await response.json();
      } catch (error) {
        console.error("❌ Lỗi API:", error);
        return null;
      }
    },

    // Lưu khi người dùng đóng tab, tắt, thoát đột ngột
    beforeUnmount() {
      window.removeEventListener("beforeunload", this.saveChat);
      this.saveChat();  // Đảm bảo gọi hàm này khi người dùng rời đi.
    },

    // Một số hàm cần chạy sao khi load trang xong
    mounted() {
      // Tải lịch sử trò chuyện từ localStorage nhưng không khôi phục cuộc trò chuyện hiện tại
      let savedHistory = localStorage.getItem("chatHistory");
      if (savedHistory) {
        this.chatHistory = JSON.parse(savedHistory); // Lưu lại lịch sử vào biến chatHistory
      }

      // Khởi tạo một cuộc trò chuyện mới
      this.newChat();

      // Sau khi tạo cuộc trò chuyện mới, cuộn xuống tin nhắn mới nhất
      this.$nextTick(this.scrollToBottom);

      // Lắng nghe sự kiện đóng trang để lưu lại dữ liệu
      window.addEventListener("beforeunload", this.saveChat);
    }
  }
}
</script>

<style scoped>
.chat-container {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  max-width: 1140px;
  width: 100%;
  border-radius: 8px;
  padding: 16px;
}

.chat-box {
  height: 300px;
  overflow-y: auto;
  border-bottom: 1px solid #ccc;
  margin-bottom: 10px;
  padding: 5px;
}

.input-area {
  display: flex;
}

.title {
  font-size: 16px;
  text-align: center;
  color: #5d5d5d;
  padding: 12px;
}

.active-chat {
  background-color: #9d9d9d !important;
  color: white !important;
}

.loading {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 10px;
}

.loading span {
  margin-left: 10px;
  font-size: 14px;
  color: #555;
}

.loading-container {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  height: 100px;
  margin-left: 16px;
}

.loading-text {
  margin-top: 10px;
  font-size: 16px;
}

.p-box {
  margin: 0;
  display: flex;
  align-items: center;
  padding-top: 6px;
}

.p-box.active {
  color: rgb(38, 166, 154);
}

.code-block {
  background-color: black !important;
}
</style>
