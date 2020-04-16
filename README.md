# vue-emoji-picker-simple
Picker for native emoji

## Usage
Copy *emoji_simple* directory to your project

Then you need to import this component to your own component were you want to use it

`import EmojiPicker from "./emoji_simple/EmojiPickerSimple"`

or register component globally

`Vue.component('EmojiPickerSimple', require('./components/emoji/EmojiPickerSimple').default)`

### Props
| Prop | Type | Default | Description |
|----|----|----|----|
| root_class | String | 'position-relative' | Class of root element |
| emojiTable | Object | import from imojiset.js | Emoji data set (json structure) |
| localStorageName | String | 'simple_emoji' | Name of local storage variable (for recently used) |
| maxRecent | Number | 10 | The maximum number of recently used emojis stored in the local storage |

### Events

***select*** - return Object of structure: {name: 'emoji_name', native: emoji_code}

## Example

```vue
  <template>
    <div class="position-relative">
      <textarea v-model="text"></textarea>
      <emoji-picker-simple root_class="emoji-picker" @select="addEmoji($event)"></emoji-picker-simple>
    </div>
  </template>

  <script>
    import EmojiPicker from "./emoji_simple/EmojiPickerSimple";
      export default {
          name: "Example",   
          components: {EmojiPicker}, 
          data(){
            return{
              text:''    
            }        
          },
          methods:{
            addEmoji(emoji){
              this.text += emoji.native
            }
          }
      }
  </script>

  <style>
    .position-relative{
      position:relative;
    }
    .emoji-picker{
      position:absolute;
      right:10px;
      top: calc(100%-30px);
    }
  </style>
```
