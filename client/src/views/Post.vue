<template>
  <div class="overlay" @click="close">
    <div class="modal" @click.stop="">
        <figure>
            <img :src="'http://192.168.2.201:4000/' + post.source" :alt="post.title" >
        </figure>
        <div class="content">
            <div class="top">
                <p>
                    <a href="#" class="author">{{user.username}}</a>
                    <span class="separator"> &middot; </span>
                    <a href="#">Seguir</a>
                </p>
                <a href="#" class="close" @click.prevent="close">
                    <svg xmlns="http://www.w3.org/2000/svg" width="32px" viewBox="0 0 512 512"><!--! Font Awesome Pro 6.2.0 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2022 Fonticons, Inc. --><path d="M175 175C184.4 165.7 199.6 165.7 208.1 175L255.1 222.1L303 175C312.4 165.7 327.6 165.7 336.1 175C346.3 184.4 346.3 199.6 336.1 208.1L289.9 255.1L336.1 303C346.3 312.4 346.3 327.6 336.1 336.1C327.6 346.3 312.4 346.3 303 336.1L255.1 289.9L208.1 336.1C199.6 346.3 184.4 346.3 175 336.1C165.7 327.6 165.7 312.4 175 303L222.1 255.1L175 208.1C165.7 199.6 165.7 184.4 175 175V175zM0 96C0 60.65 28.65 32 64 32H448C483.3 32 512 60.65 512 96V416C512 451.3 483.3 480 448 480H64C28.65 480 0 451.3 0 416V96zM48 96V416C48 424.8 55.16 432 64 432H448C456.8 432 464 424.8 464 416V96C464 87.16 456.8 80 448 80H64C55.16 80 48 87.16 48 96z"/></svg>
                </a>
            </div>
            <div class="title">
                <h1>{{post.title}}</h1>
            </div>
            <div class="comments">
                <div class="comment">
                    <div class="avatar">
                        <a href="">
                            <img src="#" alt="">
                        </a>
                    </div>
                    <div class="text">
                        <p>
                            <a href="#">{{user.username}}</a>
                            {{post.description}} 
                            <a href="#">#tag1</a>
                        </p>
                    </div>
                </div>
                <div class="comment" v-for="c in comments" v-bind:key="c">
                    <div class="avatar">
                        <a href="">
                            <img src="" alt="">
                        </a>
                    </div>
                    <div class="text">
                        <p>
                            <a href="#">{{c.username}}</a>
                                {{c.comment}} 
                            <a href="#">#tag1</a>
                        </p>
                    </div>
                </div>
            </div>
            <div class="footer">
                <div class="likes">
                    <a href="#">
                        <img  src="@/assets/img/hornero.svg" alt="Hornero"/> 
                        <span>{{post.likes}} Me gusta</span>
                    </a>
                </div>
                <form action="" @submit.prevent="newComment">
                    <input type="text" placeholder="Escribe tu comentario ..." v-model="comment.comment">
                    <input class="form-submit" type="submit" value="Publicar">
                </form>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
    import api from '@/api.js';
export default {
    props:["id"],
    data() {
        return {
            postLikes: [],
            post: {},
            user: {},
            comments:[],
            comment: {
                userId: null,
                postId: null,
                username: null,
                comment: "",

            }
        }
    },
    created(){
        this.comment.userId = this.$store.getters.loggedId
        this.comment.postId = this.id
        this.comment.username = this.$store.getters.username
        //traemos el post
        api.get(`/posts/${this.id}`).then(response => {
            this.post = response.data
            //traemos el usuario del post
            return api.get(`/users/${this.post.user_id}`)
        }).then(user =>{
            this.user = user.data
            //regresar 
            return api.get(`/posts/${this.id}/comments`)
        }).then(comments => {
            this.comments = comments.data
        })
        api.get(`/posts/likes/${this.$store.getters.loggedId}`).then(response => {
            this.postLikes = response.data
      })
    },
    methods:{
        close(){
           this.$router.push('/')
        },
        likePost(post) {
            if(this.postLikes.includes(post.post_id)){
                return
            }
            api.post(`/posts/like/${post.post_id}/${this.$store.getters.loggedId}`).then(response => {
                if(response.data.ok){
                post.likes++
                this.postLikes.push(post.post_id)
                }
            })
            },
        newComment(){
            api.post(`/posts/${this.comment.postId}/comments`, this.comment).then(response => {
                if(response.data.ok){
                    const newC = Object.assign({}, this.comment)
                    console.log(newC)
                    this.comments.push(newC)
                    this.comment.comment= ""
                }
                    
            })
        }
    }
}
</script>

<style lang="stylus" scoped>
    div.overlay 
        width 100vw
        height 100vh
        position fixed
        top 0
        left 0
        background rgba(0,0,0,0.5)
        z-index 150
        display flex
        justify-content center
        align-items center
        div.modal 
            background white
            width 80%
            height 80%
            border-radius 18px
            box-shadow 0 0 35px 0 rgba(0,0,0,0.3)
            display flex
            figure 
                margin 0
                
                img 
                    object-fit fill
                    width 100%
                    height 100%
                    padding 20px
            div.content
                width 50%
                display flex
                flex-direction column
                padding 20px
                div.top
                    display flex
                    justify-content space-between
                    align-items center
                    padding 10px 20px
                    flex 75px 0 0
                    border-bottom 1px solid #DFDFDF
                    .close
                        font-size 32px
                    .separator
                        margin 0 10px
                    .author
                        color black
                div.title
                    flex 75px 0 0
                    h1 
                        padding 20px
                        line-height 75px
                        margin 0
    div.comments
        flex 1
        overflow-y scroll
    div.comment
        display flex
        padding 20px
        margin-bottom 20px
        .avatar
            margin-right 20px
            a
                display block 
                width 40px
                height 40px
                background #EBEBEB
                border-radius 20px
        .text
            p 
                margin 0
    div.footer 
        padding 0 20px
        div.likes 
            height 50px
            line-height 50px
            img 
                margin-right 10px
            a 
                font-weight bold
        form 
            display flex
            input[type="submit"]
                width initial
                margin-left 20px
                padding 10px


                    
</style>>