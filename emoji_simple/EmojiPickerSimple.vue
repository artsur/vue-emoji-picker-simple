<template>
    <div :class="root_class">

        <button class="emoji-toggle" type="button" @click.stop="toggle">
            <svg viewBox="0 0 24 24">
                <path fill="none" d="M20,12A8,8 0 0,0 12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12M22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2A10,10 0 0,1 22,12M10,9.5C10,10.3 9.3,11 8.5,11C7.7,11 7,10.3 7,9.5C7,8.7 7.7,8 8.5,8C9.3,8 10,8.7 10,9.5M17,9.5C17,10.3 16.3,11 15.5,11C14.7,11 14,10.3 14,9.5C14,8.7 14.7,8 15.5,8C16.3,8 17,8.7 17,9.5M12,17.23C10.25,17.23 8.71,16.5 7.81,15.42L9.23,14C9.68,14.72 10.75,15.23 12,15.23C13.25,15.23 14.32,14.72 14.77,14L16.19,15.42C15.29,16.5 13.75,17.23 12,17.23Z" />
            </svg>
        </button>

        <div v-if="display"  v-click-outside="hide" class="emoji-picker-dialog">
            <div class="emoji-list">
                <div v-if="recent && !search" class="emoji-category">
                    <div class="category-title">Recently Used</div>
                    <div class="emoji-icons">
                        <div class="emoji-icon"
                             v-for="(emoji, emojiName) in recent"
                             :key="emojiName"
                             @click="insert(emoji,emojiName)"
                             :title="emojiName"
                        >{{ emoji }}</div>
                    </div>
                </div>
                <div v-for="(emojiGroup, category) in emojiset" :key="category" class="emoji-category">
                    <div class="category-title">{{ category }}</div>
                    <div class="emoji-icons">
                        <div class="emoji-icon"
                             v-for="(emoji, emojiName) in emojiGroup"
                             :key="emojiName"
                             @click="insert(emoji,emojiName)"
                             :title="emojiName"
                        >{{ emoji }}</div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import emojis from './emojiset'
    export default {
        props: {
            root_class:{type:String,default:'position-relative'}, //Class of root element
            emojiTable:{type: Object,default(){return emojis}}, //Emoji set
            localStorageName: {type:String,default:'simple_emoji'}, //Name for local storage (recent use)
            maxRecent:{type:Number,default:10} //The maximum number of recent emojis stored in the local storage
        },
        data() {
            return {
                display: false,
                search:null,
                recent:null,
            }
        },
        watch:{
            display(val){
                if(!val){
                    this.search = null;
                }
            }
        },
        computed: {
            emojiset() {
                if (this.search) {
                    let obj = {};
                    for (let category in this.emojiTable) {
                        obj[category] = {};
                        for (let emoji in this.emojiTable[category]) {
                            if (new RegExp(`.*${this.search}.*`).test(emoji)) {
                                obj[category][emoji] = this.emojiTable[category][emoji]
                            }
                        }
                        if (Object.keys(obj[category]).length === 0) {
                            delete obj[category];
                        }
                    }
                    return obj;
                }
                return this.emojiTable;
            },
            stored(){
                if (localStorage.getItem(this.localStorageName)) {
                    try {
                        return  JSON.parse(localStorage.getItem(this.localStorageName));
                    } catch(e) {
                        localStorage.removeItem(this.localStorageName);
                        return [];
                    }
                }
                return[];
            }
        },
        methods: {
            insert(emoji,emojiName) {
                this.$emit('select', {native:emoji,name:emojiName});
                this.saveToLocalStorage(emojiName);
            },
            toggle(e) {
                if(this.display){
                    this.hide();
                }else{
                    this.getRecent(this.stored);
                    this.display = true;
                }
            },
            hide() {
                this.getRecent(this.stored);
                this.display = false;
            },
            /*escape(e) {
                if (this.display === true && e.keyCode === 27) {
                    this.hide();
                }
            },*/
            saveToLocalStorage(emojiName){
                if(this.stored){
                    let recent = this.stored;
                    recent.splice(0,0,emojiName);
                    let unic = [];
                    for (let str of this.stored) {
                        if (!unic.includes(str)) {
                            unic.push(str);
                        }
                    }
                    if(unic.length > this.maxRecent){
                        unic = unic.slice(0,this.maxRecent);
                    }
                    localStorage.setItem(this.localStorageName, JSON.stringify(unic));
                }
            },
            getRecent(stored){
                if(!stored) stored = this.stored;
                setTimeout(()=>{
                    let res = {};
                    if(stored && stored.length>0){
                        stored.forEach((el)=>{
                            for (let category in this.emojiTable) {
                                for (let emoji in this.emojiTable[category]) {
                                    if(el === emoji) res[el] = this.emojiTable[category][emoji];
                                }
                            }
                        });
                    }
                    this.recent = res;
                },50);
            }
        },
        directives: {
            'click-outside': {
                bind(el, binding, vNode) {
                    if (typeof binding.value !== 'function') {
                        return;
                    }
                    let bubble = binding.modifiers.bubble;
                    let handler = (e) => {
                        if (bubble || (! el.contains(e.target) && el !== e.target)) {
                            binding.value(e);
                        }
                    };
                    el.__vueClickOutside__ = handler;
                    document.addEventListener('click', handler);
                },
                unbind(el, binding) {
                    document.removeEventListener('click', el.__vueClickOutside__);
                    el.__vueClickOutside__ = null;
                },
            },
        },
        /*mounted() {
            document.addEventListener('keyup', this.escape);
        },
        destroyed() {
            document.removeEventListener('keyup', this.escape);
        },*/
    }
</script>

<style lang="scss">
    .emoji-toggle{
        position:static;
        border:0;
        background: transparent;
        outline: none !important;
        padding:0;
        height:24px;
        width:24px;
        cursor:pointer;
        svg{
            height:24px;
            width:24px;
        }
        path{fill:#888888;}
        &:hover{
            transform: scale(1.1);
            path{
                fill: #195aa7;
            }
        }
    }

    .emoji-picker-dialog {
        position: relative;
        z-index: 2;
        border: 1px solid #ccc;
        width: 30rem;
        height: 20rem;
        max-width:100%;
        max-height: 40vh;
        overflow-y: auto;
        padding: 0.5rem;
        box-sizing: border-box;
        border-radius: 0.5rem;
        background: #fff;
        box-shadow: 3px 3px 15px rgba(0,0,0,0.2);

        .category-title {
            display:block;
            border-bottom: 1px solid #f2f2f2;
            margin-top: 0.75rem;
            margin-bottom:0.25rem;
            color: #888888;
            text-transform: uppercase;
            font-size: 0.8rem;
            cursor: default;
        }

        .emoji-icons {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            &:after {
                content: "";
                flex: auto;
            }

            .emoji-icon{
                display: inline-block;
                margin: 0.2rem;
                padding: 0.2rem;
                cursor: pointer;
                border-radius: 50%;
                font-size: 1.6rem;
                line-height:1.6rem;
                &:hover {
                    background: #eeeeee;
                    transform: scale(1.2);
                }
            }
        }
    }
</style>
