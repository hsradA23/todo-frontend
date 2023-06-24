<script>
    import { DateInput } from "date-picker-svelte";
    import InPlaceEdit from "./lib/InPlaceEdit.svelte";
    import { onMount } from "svelte";
    import TailwindCss from "./Tailwind.svelte";
    import DelIcon from "./assets/delete.svelte";
    import { notifications } from "./lib/notifications.js";
    import Toast from "./lib/Toast.svelte";
    import Select from 'svelte-select';

    let newtodo = "";
    let date = new Date();

    let user = "";
    let token = "";

    let loggedIn = false;

    // Sample tasks, to make debugging easier
    // let tasks = [
    //     [1, 0, "Eat fruit", "2008-11-11"],
    //     [2, 0, "Sleep well", "2008-11-11"],
    //     [3, 0, "drink water", "2023-06-22T15:04:42.180Z"],
    //     [4, 0, "have dinner and brush teeth", "2023-06-22T16:52:26.686Z"],
    //     [5, 0, "sleep", "2023-06-22T16:57:16.055Z"],
    //     [6, 0, "drink ", "2023-06-22T16:58:31.949Z"],
    // ];
    let items = ['Pending', 'Done'];
    let doneFilter = null;
    let searchTerm = ''
    let tasks = [];
    let arr = tasks.map((t) => t[1] == "1");
    let dates = tasks.map((t) => t[3].toString().slice(8,10) + '-'+ t[3].toString().slice(5,7) + '-' + t[3].toString().slice(0,4));
    // let dates = tasks.map((t) => t[3].toString());
    console.log(dates)


    onMount(async () => {
        //getTasks();
    });

    function login() {
        fetch("/login", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                user,
                token,
            }),
        })
            .then((r) => r.text())
            .then((t) => {
                console.log(t);
                loggedIn = t == "Successful";
                if (loggedIn) {
                    notifications.success("Logged in Successfully", 1000);
                    getTasks();
                } else {
                    notifications.danger("Invalid Username or Password", 2000);
                }
            })
            .catch((error) => {
                console.error("Error:", error);
            });
    }
    function logout() {
        user = "";
        token = "";
        loggedIn = false;
    }

    function register() {
        fetch("/register", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                user,
                token,
            }),
        })
            .then((r) => r.text())
            .then((t) => {
                console.log(t);
                loggedIn = t == "Successful";
                if (loggedIn) {
                    notifications.success("Created New Account", 1000);
                    getTasks();
                } else {
                    notifications.danger("User already exists.", 2000);
                }
            })
            .catch((error) => {
                console.error("Error:", error);
            });
    }

    // /api/todo/add
    const addTask = () => {
        fetch("/api/todo/add", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                user,
                token,
                newtodo,
                date,
            }),
        })
            .then((response) => {
                getTasks();
                newtodo = "";
            })
            .catch((error) => {
                // Handle any errors
                console.error("Error:", error);
            });
    };

    const getTasks = () => {
        fetch("/api/todo/get", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                user,
                token,
                doneFilter : doneFilter !=null ? doneFilter.label:'ALL',
                searchTerm
            }),
        })
            .then((response) => response.json())
            .then((data) => {
                // Handle the response data
                console.log(data);
                tasks = data;
                arr = tasks.map((t) => t[1] == "1");
                dates = dates = tasks.map((t) => t[3].toString().slice(8,10) + '-'+ t[3].toString().slice(5,7) + '-' + t[3].toString().slice(0,4));

                console.log(arr);
            })
            .catch((error) => {
                // Handle any errors
                console.error("Error:", error);
            });
    };

    function submit(field, tid) {
        return ({ detail: newValue }) => {
            console.log(
                `updated ${field}, new value is: "${newValue}" id ${tid}`
            );

            fetch("/api/todo/update", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                    user,
                    token,
                    tid,
                    newValue,
                }),
            })
                .then((response) => response.json())
                .then((data) => {
                    // Handle the response data
                    console.log(data);
                    tasks = data;
                })
                .catch((error) => {
                    // Handle any errors
                    console.error("Error:", error);
                });
        };
    }

    function done(tid) {
        fetch("/api/todo/done", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                tid,
            }),
        }).then((r) => getTasks());
    }

    function deleteTask(tid) {
        fetch("/api/todo/delete", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                tid,
            }),
        }).then((r) => getTasks());
    }
</script>

<TailwindCss />

<main>
    <div class="w-full h-full flex flex-col items-center">
        {#if loggedIn}
            <div
                class="w-9/12 bg-blue-500 flex justify-center items-center gap-10 p-6 rounded-b-xl"
            >
                <input
                    bind:value={newtodo}
                    on:keypress={(e) => {
                        if (e.key === "Enter") addTask();
                    }}
                    placeholder="Enter a task"
                    class="rounded-md bg-gray-100 px-3 py-1.5 border-2 border-gray-200 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-blue-700"
                />
                <DateInput bind:value={date} format="dd-MM-yyyy" class="" />
                <button
                    class="bg-blue-100 p-1.5 rounded-md hover:bg-green-400 transition-colors"
                    on:click={addTask}>Add Task</button
                >

                <button
                    class="bg-blue-100 p-1.5 rounded-md hover:bg-red-500 transition-colors ml-10"
                    on:click={logout}>LogOut</button
                >
            </div>
            <!-- 
        <button on:click={getTasks}>Get Data</button> -->
            <div class="flex flex-col align-start items-start w-5/12">
                <div
                    class="w-full bg-blue-500 flex justify-center items-center gap-10 px-6 py-5 rounded-b-xl mb-7"
                >
                    <input
                        placeholder="Search"
                        bind:value={searchTerm}
                        class="flex-grow rounded-md bg-gray-100 p-2 border-2 border-gray-200 text-gray-700 leading-tight focus:outline-none focus:bg-white focus:border-blue-700"
                    />
                    <Select
                        {items}
                        bind:value={doneFilter}
                        class="grow-0"
                        placeholder="All"
                    />
                    <button
                        class="bg-blue-100 p-2 px-4 rounded-md hover:bg-green-400 transition-colors"
                        on:click={getTasks}>Filter</button
                    >
                </div>
                {#each tasks as task, idx}
                    <div
                        class="flex flex-row items-center gap-2 min-w-full odd:bg-blue-100 even:bg-blue-200 rounded-md p-3 m-1 hover:bg-blue-400 transition-colors"
                    >
                        <input
                            type="checkbox"
                            bind:checked={arr[idx]}
                            on:change={() => done(task[0])}
                            class="w-5 h-5"
                        />
                        <div class="p-2 flex-grow focus:bg-white  text-md font-medium">
                            <InPlaceEdit
                            bind:value={task[2]}
                            on:submit={submit(task[2], task[0])}
                            />
                        </div>
                        <div class="text-blue-500 text-sm font-extrabold">{dates[idx]}</div>
                        <div
                            on:click={() => deleteTask(task[0])}
                            class="p-2 rounded-md hover:bg-red-400 transition-colors"
                        >
                            <DelIcon />
                        </div>
                    </div>
                {/each}
            </div>
        {:else}
            <div class="h-screen flex justify-center items-center">
                <div
                    class="bg-blue-100 p-5 rounded-xl flex flex-col gap-5 text-blue"
                >
                    <div class="text-gray-500 font-bold px-7 text-xl">
                        Login Or Register
                    </div>
                    <div class="flex flex-row justify-between items-center">
                        <label for="username" class="mx-7">Username</label>
                        <input
                            type="text"
                            name="username"
                            class="mx-7 p-2 rounded-md"
                            bind:value={user}
                        />
                    </div>
                    <div class="flex flex-row justify-between items-center">
                        <label for="password" class="mx-7">Password</label>
                        <input
                            type="password"
                            name="password"
                            class="mx-7 p-2 rounded-md"
                            bind:value={token}
                        />
                    </div>
                    <div class="flex flex-row justify-between">
                        <button
                            class="mx-7 bg-blue-400 p-2 rounded-md hover:bg-blue-500 transition-colors text-white font-bold"
                            on:click={login}>LogIn</button
                        >
                        <button
                            class="mx-7 bg-green-500 p-2 rounded-md hover:bg-green-600 transition-colors text-white font-bold"
                            on:click={register}>Register</button
                        >
                    </div>
                </div>
            </div>
        {/if}
    </div>
</main>
<Toast />

<style>
</style>
