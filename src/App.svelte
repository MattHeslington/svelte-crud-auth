<script>
    // DEPLOY TO NETLIFY 22:44

    //init
    let userObject = null;
    const userbase = window.userbase;
    let authPromise = userbase.init({ appId: '68fb3077-4ee3-46bb-aed2-3e977698cb98' })
        .then(({user}) => userObject = user)

    //auth
    let username, password;
    const signIn = () => authPromise = userbase.signIn({username, password}).then(user => userObject = user);
    const signUp = () => authPromise = userbase.signUp({username, password}).then(user => userObject = user);
    const signOut = () => authPromise = userbase.signOut().then(() => userObject = null)

    //db
    let todoPromise;
    let todos = [], newTodo = '';
    const databaseName = 'todos';
    function changeHandler(items) {todos = items}
    $: if (userObject) todoPromise = userbase.openDatabase({databaseName, changeHandler})
    function addTodo() {
        todoPromise = userbase.insertItem({databaseName, item: newTodo})
        newTodo = '';
    }

    function deleteTodo(itemId) {
        todoPromise = userbase.deleteItem({databaseName, itemId});
    }

    function updateTodo(index) {
        const {item, itemId} = todos[index];
        todoPromise = userbase.updateItem({databaseName, itemId, item});
    }

</script>

{#await authPromise}Loading...{:then _}
{#if !userObject}
<form>
    <label for="username">Username</label>
    <input id="username" type="text" bind:value={username}><br>
    <label for="password">password</label>
    <input id="password" type="password" bind:value={password}><br>
    <button on:click={signIn} type="button">Sign In</button>
    <button on:click={signUp} type="button">Sign Up</button>
</form>
{:else}
    <h3>Hello {userObject.username}</h3>
    <button on:click={signOut} style="position:absolute; top:10px; right:10px">Sign Out</button>
    {#await todoPromise}Loading ToDos{:then _}
        <label for="new-todo">New Todo</label>
        <input id="new-todo" type="text" bind:value={newTodo}/>
        <button on:click={addTodo}>Add Todo</button><br>

        {#each todos as {item, itemId}, index}
            <input type="text" bind:value={todos[index].item} on:blur={() => updateTodo(index)}/>
            <button on:click={() => deleteTodo(itemId)}>X</button><br>
        {/each}
    {:catch error}Error...{error}{/await}
{/if}

{:catch error}Error...{error} {/await}