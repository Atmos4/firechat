<script lang="ts">
	import { signInWithPopup, signOut, type User as FUser } from 'firebase/auth';
	import {
		addDoc,
		collection,
		doc,
		getDoc,
		orderBy,
		query,
		serverTimestamp,
		setDoc
	} from 'firebase/firestore';
	import { GoogleAuthProvider } from 'firebase/auth';
	import { Collection, FirebaseApp, User } from 'sveltefire';

	import { initializeApp } from 'firebase/app';
	import { getFirestore } from 'firebase/firestore';
	import { getAuth } from 'firebase/auth';

	// Initialize Firebase
	const app = initializeApp({
		apiKey: 'AIzaSyC24TM3Ua6lucNAsmQL1rtd6zEUKTxXQss',
		authDomain: 'svetlekit-firebase-test.firebaseapp.com',
		projectId: 'svetlekit-firebase-test',
		storageBucket: 'svetlekit-firebase-test.appspot.com',
		messagingSenderId: '1093165050781',
		appId: '1:1093165050781:web:9576295b3a9b28cd8d9d74'
	});

	const firestore = getFirestore(app);
	const auth = getAuth(app);

	const provider = new GoogleAuthProvider();

	async function signIn() {
		const { user } = await signInWithPopup(auth, provider);
		const existingUser = await getDoc(doc(firestore, 'users', user.uid));
		if (!existingUser.exists()) {
			await setDoc(doc(firestore, 'users', user.uid), {
				name: user.displayName
			});
		}
	}

	let value: string = '';

	async function addMessage(user: FUser) {
		await addDoc(collection(firestore, `messages`), {
			content: value,
			created: serverTimestamp(),
			userId: user.uid
		});
		value = '';
	}

	$: getMessages = () => {
		const q = query(collection(firestore, `messages`), orderBy('created', 'desc'));
		return q;
	};

	function getTime(timestamp: number) {
		const date = new Date(timestamp * 1000);
		return `${date.getHours()}:${date.getMinutes()}`;
	}
</script>

<FirebaseApp {auth} {firestore}>
	<User let:user>
		<nav class="container-fluid">
			<ul>
				<li>
					<p>Hello {user.displayName}</p>
				</li>
			</ul>
			<ul>
				<li><button on:click={() => signOut(auth)} class="secondary">Sign out</button></li>
			</ul>
		</nav>
		<main class="container">
			<form on:submit|preventDefault={() => addMessage(user)} role="group">
				<input type="text" placeholder="Message" bind:value />
				<button>Send</button>
			</form>
			<Collection ref={getMessages()} let:data={posts}>
				<section class="messages">
					{#each posts as post (post.id)}
						<div class={post.userId == user.uid ? 'message me' : 'message other'}>
							<div class="content">{post.content}</div>
							<div class="info">
								{getTime(post.created)}
							</div>
						</div>
					{/each}
				</section>
			</Collection>
		</main>

		<main slot="signedOut" class="container">
			<article>
				<p>Sign in to do stuff</p>
				<button on:click={() => signIn()}>Sign in</button>
			</article>
		</main>
	</User>
</FirebaseApp>

<style>
	.messages {
		display: flex;
		flex-direction: column;
	}
	.message {
		margin: 0.5em;
		width: fit-content;
		display: flex;
		align-items: center;
		gap: 0.5em;
	}
	.message.me {
		align-self: flex-end;
		flex-direction: row-reverse;
	}
	.message > .content {
		padding: 0.8em 1em;
		background-color: rgb(0, 102, 102);
		border-radius: 1em 1em 1em 0;
	}
	.message.me > .content {
		background-color: rgb(0, 102, 42);
		border-radius: 1em 1em 0 1em;
	}
	.message > .info {
		font-size: 0.75em;
		display: flex;
		gap: 1em;
	}
</style>
