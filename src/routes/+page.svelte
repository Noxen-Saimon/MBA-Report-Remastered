<script lang="ts">
	/* eslint-disable svelte/prefer-svelte-reactivity */
	import './page.css';
	import { createClient } from '@libsql/client';

	const title = 'MBA REPORT - Group 1';

	// ========== SENSITIVE (dari .env) ==========
	const TURSO_URL = import.meta.env.VITE_TURSO_URL;
	const TURSO_TOKEN = import.meta.env.VITE_TURSO_TOKEN;

	// ========== CONFIG (penting tapi bisa diubah) ==========
	const HOST_CONFIG = { START_DATE_HOST: '2026-08-09', START_INDEX_HOST: 1 };
	const CONFIG = { START_DATE: '2026-05-03', ANCHOR_BOOK: '2 Taw', ANCHOR_CH: 12, UNLOCK_HOUR: 19 };
	const THEME_KEY = 'mba_theme_mode';

	const BIBLE = [
		{ n: 'Kej', c: 50 },
		{ n: 'Kel', c: 40 },
		{ n: 'Im', c: 27 },
		{ n: 'Bil', c: 36 },
		{ n: 'Ul', c: 34 },
		{ n: 'Yosua', c: 24 },
		{ n: 'Hak', c: 21 },
		{ n: 'Rut', c: 4 },
		{ n: '1 Sam', c: 31 },
		{ n: '2 Sam', c: 24 },
		{ n: '1 Raj', c: 22 },
		{ n: '2 Raj', c: 25 },
		{ n: '1 Taw', c: 29 },
		{ n: '2 Taw', c: 36 },
		{ n: 'Ezr', c: 10 },
		{ n: 'Neh', c: 13 },
		{ n: 'Est', c: 10 },
		{ n: 'Ayb', c: 42 },
		{ n: 'Mzm', c: 150 },
		{ n: 'Ams', c: 31 },
		{ n: 'Pkh', c: 12 },
		{ n: 'Kid', c: 8 },
		{ n: 'Yes', c: 66 },
		{ n: 'Yer', c: 52 },
		{ n: 'Rat', c: 5 },
		{ n: 'Yeh', c: 48 },
		{ n: 'Dan', c: 12 },
		{ n: 'Hos', c: 14 },
		{ n: 'Yl', c: 3 },
		{ n: 'Am', c: 9 },
		{ n: 'Ob', c: 1 },
		{ n: 'Yun', c: 4 },
		{ n: 'Mi', c: 7 },
		{ n: 'Nah', c: 3 },
		{ n: 'Hab', c: 3 },
		{ n: 'Zef', c: 3 },
		{ n: 'Hag', c: 2 },
		{ n: 'Za', c: 14 },
		{ n: 'Mal', c: 4 },
		{ n: 'Matius', c: 28 },
		{ n: 'Mrk', c: 16 },
		{ n: 'Luk', c: 24 },
		{ n: 'Yoh', c: 21 },
		{ n: 'Kis', c: 28 },
		{ n: 'Rom', c: 16 },
		{ n: '1 Kor', c: 16 },
		{ n: '2 Kor', c: 13 },
		{ n: 'Gal', c: 6 },
		{ n: 'Ef', c: 6 },
		{ n: 'Flp', c: 4 },
		{ n: 'Kol', c: 4 },
		{ n: '1 Tes', c: 5 },
		{ n: '2 Tes', c: 3 },
		{ n: '1 Tim', c: 6 },
		{ n: '2 Tim', c: 4 },
		{ n: 'Tit', c: 3 },
		{ n: 'Flm', c: 1 },
		{ n: 'Ibr', c: 13 },
		{ n: 'Yak', c: 5 },
		{ n: '1 Pet', c: 5 },
		{ n: '2 Pet', c: 3 },
		{ n: '1 Yoh', c: 5 },
		{ n: '2 Yoh', c: 1 },
		{ n: '3 Yoh', c: 1 },
		{ n: 'Yud', c: 1 },
		{ n: 'Why', c: 22 }
	];

	// ========== DAILY EMOJI ==========

	const DAILY_EMOJI_POOL = [
		'🔥',
		'🌟',
		'✨',
		'🌈',
		'☀️',
		'🌙',
		'💫',
		'🎯',
		'🚀',
		'💪',
		'🙌',
		'👏',
		'❤️',
		'💚',
		'💙',
		'💜',
		'🧡',
		'💛',
		'🤍',
		'😎',
		'😄',
		'😁',
		'😊',
		'🥳',
		'😇',
		'🤩',
		'🫶',
		'🙏',
		'📖',
		'📚',
		'✝️',
		'🕊️',
		'🌿',
		'🍀',
		'🌻',
		'🌺',
		'🌸',
		'🌼',
		'🍃',
		'☁️',
		'🌤️',
		'🌍',
		'🎉',
		'🎊',
		'🏆',
		'💎',
		'⚡',
		'🦋',
		'🐳',
		'🐝',
		'🦁',
		'🐻',
		'🦊',
		'🐼',
		'🐬',
		'🌊',
		'🍎',
		'🍇',
		'🍀',
		'🎵',
		'🎶'
	];

	// ========== HELPER ==========
	const pad = (n: number) => String(n).padStart(2, '0');

	// ========== TYPES ==========
	type Member = { name: string; status: 0 | 1 | 2 };
	type BibleReading = { n: string; c: number };
	type ReportRow = { date: string; status_data: string | number[] };
	type UserSession = { name: string; hostPeriod: string };

	// ========== DATABASE ==========
	const db = createClient({ url: TURSO_URL, authToken: TURSO_TOKEN });
	let dbStatus = $state<'connecting' | 'online' | 'saving' | 'error'>('connecting');
	let dbHistory = $state<Record<string, number[]>>({});

	// ========== STATE ==========
	let now = $state(new Date());
	let selectedDate = $state(new Date().toISOString().split('T')[0]);
	let darkMode = $state(true);
	let godMode = $state(false);

	// ========== USER SESSION ==========
	const USER_SESSION_KEY = 'mba_user_session';

	let currentUser = $state<string | null>(null);
	let selectedUser = $state<string | null>(null);
	let showUserSelector = $state(true);
	let checkedHostPeriod = $state('');

	let showIdentityConfirm = $state(false);

	let toastMsg = $state('');
	let toastShow = $state(false);
	let toastTimer: ReturnType<typeof setTimeout> | undefined;

	let members = $state<Member[]>([
		{ name: 'Chui Chui', status: 0 },
		{ name: 'Emi', status: 0 },
		{ name: 'GI. Aning', status: 1 },
		{ name: 'Gunawan', status: 2 },
		{ name: 'Hans Edgar', status: 0 },
		{ name: 'John Zechariah', status: 0 },
		{ name: 'Lena Hadi', status: 0 },
		{ name: 'Metta', status: 0 },
		{ name: 'Riany', status: 0 },
		{ name: 'Rusjiati', status: 0 },
		{ name: 'Shimu Ai Fie', status: 0 },
		{ name: 'Siok Lian', status: 0 },
		{ name: 'Steven B', status: 0 },
		{ name: 'Sundari', status: 0 },
		{ name: 'Vivi', status: 0 },
		{ name: 'Wenny', status: 0 },
		{ name: 'Yoenita', status: 0 }
	]);

	// ========== DERIVED ==========
	const isCurrentHost = $derived(currentUser !== null && currentUser === hostName);
	const selectedUserIsHost = $derived(selectedUser !== null && selectedUser === hostName);

	const dailyEmoji = $derived(DAILY_EMOJI_POOL[hashDate(selectedDate) % DAILY_EMOJI_POOL.length]);

	const clockText = $derived(
		`${pad(now.getHours())}:${pad(now.getMinutes())}:${pad(now.getSeconds())}`
	);
	const dateText = $derived(
		now
			.toLocaleDateString('id-ID', {
				weekday: 'long',
				day: 'numeric',
				month: 'long',
				year: 'numeric'
			})
			.toUpperCase()
	);
	const previewUnlocked = $derived(now.getHours() >= CONFIG.UNLOCK_HOUR);

	const hostName = $derived.by(() => {
		const anchor = new Date(HOST_CONFIG.START_DATE_HOST);
		const target = new Date(selectedDate);
		anchor.setHours(12, 0, 0, 0);
		target.setHours(12, 0, 0, 0);
		const diffDays = Math.round((target.getTime() - anchor.getTime()) / (1000 * 60 * 60 * 24));
		const weeks = Math.floor(diffDays / 7);
		let index = HOST_CONFIG.START_INDEX_HOST + weeks;
		index = ((index % members.length) + members.length) % members.length;
		return members[index]?.name ?? '--';
	});

	const hostWeekText = $derived.by(() => {
		const d = new Date(selectedDate);
		const start = new Date(d);
		const day = start.getDay();
		const diff = day === 0 ? 6 : day - 1;
		start.setDate(start.getDate() - diff);
		const end = new Date(start);
		end.setDate(start.getDate() + 6);
		const fmt = (date: Date) =>
			date.toLocaleDateString('id-ID', { day: 'numeric', month: 'short' });
		return `${fmt(start)} – ${fmt(end)}`;
	});

	const fullBible = $derived.by(() => {
		const arr: BibleReading[] = [];
		BIBLE.forEach((b) => {
			for (let i = 1; i <= b.c; i++) arr.push({ n: b.n, c: i });
		});
		return arr;
	});

	const todayReading = $derived.by(() => {
		const anchor = new Date(CONFIG.START_DATE);
		const d = new Date(selectedDate);
		anchor.setHours(12, 0, 0, 0);
		d.setHours(12, 0, 0, 0);
		const diff = Math.round((d.getTime() - anchor.getTime()) / (1000 * 60 * 60 * 24));
		const aIdx = fullBible.findIndex((x) => x.n === CONFIG.ANCHOR_BOOK && x.c === CONFIG.ANCHOR_CH);
		let start = (aIdx + diff * 3) % fullBible.length;
		if (start < 0) start += fullBible.length;
		const readings = [
			fullBible[start % fullBible.length],
			fullBible[(start + 1) % fullBible.length],
			fullBible[(start + 2) % fullBible.length]
		];
		return formatReading(readings);
	});

	const tomorrowReading = $derived.by(() => {
		const d = new Date(selectedDate);
		d.setDate(d.getDate() + 1);
		const anchor = new Date(CONFIG.START_DATE);
		anchor.setHours(12, 0, 0, 0);
		d.setHours(12, 0, 0, 0);
		const diff = Math.round((d.getTime() - anchor.getTime()) / (1000 * 60 * 60 * 24));
		const aIdx = fullBible.findIndex((x) => x.n === CONFIG.ANCHOR_BOOK && x.c === CONFIG.ANCHOR_CH);
		let start = (aIdx + diff * 3) % fullBible.length;
		if (start < 0) start += fullBible.length;
		const readings = [
			fullBible[start % fullBible.length],
			fullBible[(start + 1) % fullBible.length],
			fullBible[(start + 2) % fullBible.length]
		];
		return formatReading(readings);
	});

	const isReadOnly = $derived.by(() => {
		if (godMode) return false;
		const sel = new Date(selectedDate);
		const today = new Date();
		return sel.toDateString() !== today.toDateString();
	});

	const bannerText = $derived.by(() => {
		if (godMode) return 'ADMIN MODE';
		return isReadOnly ? 'HISTORY · READ ONLY' : 'READY TO BE REPORTED';
	});

	const bannerClass = $derived.by(() => {
		if (godMode) return 'status-banner banner-live';
		return isReadOnly ? 'status-banner banner-history' : 'status-banner banner-live';
	});

	const tomorrowPreview = $derived.by(() => {
		const d = new Date(selectedDate);
		d.setDate(d.getDate() + 1);
		const dateFormatted = d
			.toLocaleDateString('id-ID', {
				weekday: 'long',
				day: 'numeric',
				month: 'long',
				year: 'numeric'
			})
			.toUpperCase();
		return (
			`MENU BACAAN\n\n${dateFormatted}\n\nBacaan: ${tomorrowReading}\n\n` +
			`YESAYA 40:8 (TB2).\nRumput menjadi kering, bunga menjadi layu, tetapi Firman ALLAH kita TETAP untuk selama-lamanya.\n\n` +
			`TUHAN YESUS MEMBERKATI.`
		);
	});

	// ========== EFFECTS ==========
	$effect(() => {
		const id = setInterval(() => (now = new Date()), 1000);
		return () => clearInterval(id);
	});

	$effect(() => {
		const currentPeriod = getHostPeriodStart(now);

		if (currentPeriod !== checkedHostPeriod) {
			checkedHostPeriod = currentPeriod;

			checkUserSession();
		}
	});

	$effect(() => {
		const id = setInterval(() => (now = new Date()), 1000);
		return () => clearInterval(id);
	});

	$effect(() => {
		document.body.classList.toggle('dark-mode', darkMode);
		document.documentElement.style.colorScheme = darkMode ? 'dark' : 'light';
		localStorage.setItem(THEME_KEY, darkMode ? 'dark' : 'light');
	});

	$effect(() => {
		loadDataFromDB();
	});

	$effect(() => {
		loadStatusFromHistory();
	});

	// ========== FUNCTIONS ==========
	function getHostPeriodStart(date: Date): string {
		const d = new Date(date);
		d.setHours(12, 0, 0, 0);
		d.setDate(d.getDate() - d.getDay());

		const year = d.getFullYear();
		const month = String(d.getMonth() + 1).padStart(2, '0');
		const day = String(d.getDate()).padStart(2, '0');

		return `${year}-${month}-${day}`;
	}

	function confirmUser() {
		if (!selectedUser) {
			showToast('Pilih nama terlebih dahulu');
			return;
		}

		// Buka tahap konfirmasi identitas
		showIdentityConfirm = true;
	}

	function completeUserLogin() {
		if (!selectedUser) return;

		const session: UserSession = {
			name: selectedUser,
			hostPeriod: getHostPeriodStart(new Date())
		};

		localStorage.setItem(USER_SESSION_KEY, JSON.stringify(session));

		currentUser = selectedUser;

		showIdentityConfirm = false;
		showUserSelector = false;

		showToast(`Masuk sebagai ${selectedUser}`);
	}

	function cancelIdentityConfirm() {
		showIdentityConfirm = false;
	}

	function checkUserSession() {
		const savedSession = localStorage.getItem(USER_SESSION_KEY);

		// Belum pernah login
		if (!savedSession) {
			currentUser = null;
			selectedUser = null;
			showUserSelector = true;
			return;
		}

		try {
			const session = JSON.parse(savedSession) as UserSession;

			// Pastikan user masih ada dalam daftar member
			const userExists = members.some((member) => member.name === session.name);

			if (!userExists) {
				localStorage.removeItem(USER_SESSION_KEY);

				currentUser = null;
				selectedUser = null;
				showUserSelector = true;

				return;
			}

			const currentPeriod = getHostPeriodStart(new Date());

			// Periode host sudah berganti
			if (session.hostPeriod !== currentPeriod) {
				localStorage.removeItem(USER_SESSION_KEY);

				currentUser = null;
				selectedUser = null;
				showUserSelector = true;

				showToast('Periode host telah berganti');

				return;
			}

			// Session masih valid
			currentUser = session.name;
			selectedUser = session.name;
			showUserSelector = false;
		} catch {
			// Session rusak / invalid
			localStorage.removeItem(USER_SESSION_KEY);

			currentUser = null;
			selectedUser = null;
			showUserSelector = true;
		}
	}

	function logoutUser() {
		localStorage.removeItem(USER_SESSION_KEY);

		currentUser = null;
		selectedUser = null;
		showUserSelector = true;

		showToast('Kamu telah logout');
	}

	function getReadingForDate(dateString: string): string {
		const anchor = new Date(CONFIG.START_DATE);
		const target = new Date(dateString);

		anchor.setHours(12, 0, 0, 0);
		target.setHours(12, 0, 0, 0);

		const diff = Math.round((target.getTime() - anchor.getTime()) / (1000 * 60 * 60 * 24));

		const aIdx = fullBible.findIndex((x) => x.n === CONFIG.ANCHOR_BOOK && x.c === CONFIG.ANCHOR_CH);

		let start = (aIdx + diff * 3) % fullBible.length;

		if (start < 0) {
			start += fullBible.length;
		}

		const readings = [
			fullBible[start % fullBible.length],
			fullBible[(start + 1) % fullBible.length],
			fullBible[(start + 2) % fullBible.length]
		];

		return formatReading(readings);
	}
	function formatReading(arr: BibleReading[]): string {
		const groups: { book: string; chs: number[] }[] = [];
		arr.forEach((p) => {
			const last = groups[groups.length - 1];
			if (last && last.book === p.n) last.chs.push(p.c);
			else groups.push({ book: p.n, chs: [p.c] });
		});
		return groups
			.map((x) => {
				const chapter = x.chs.length > 1 ? `${x.chs[0]}-${x.chs[x.chs.length - 1]}` : x.chs[0];
				return `${x.book.toUpperCase()} ${chapter}`;
			})
			.join(' & ');
	}

	function hashDate(date: string): number {
		let hash = 2166136261;

		for (let i = 0; i < date.length; i++) {
			hash ^= date.charCodeAt(i);
			hash = Math.imul(hash, 16777619);
		}

		return hash >>> 0;
	}

	async function loadDataFromDB() {
		dbStatus = 'connecting';
		try {
			const rs = await db.execute('SELECT date, status_data FROM mba_reports');
			rs.rows.forEach((row) => {
				const r = row as unknown as ReportRow;
				dbHistory[r.date] =
					typeof r.status_data === 'string' ? JSON.parse(r.status_data) : r.status_data;
			});
			dbStatus = 'online';
		} catch (err) {
			console.error('DB Error:', err);
			dbStatus = 'error';
		}
	}

	async function saveToDB() {
		dbStatus = 'saving';
		const dateKey = selectedDate;
		const currentData = members.map((m) => m.status);
		try {
			await db.execute({
				sql: `INSERT INTO mba_reports (date, status_data) VALUES (?, ?) ON CONFLICT(date) DO UPDATE SET status_data = excluded.status_data`,
				args: [dateKey, JSON.stringify(currentData)]
			});
			dbHistory[dateKey] = [...currentData];
			dbStatus = 'online';
			showToast('Tersimpan');
		} catch (err) {
			console.error('Save Error:', err);
			dbStatus = 'error';
			showToast('Gagal menyimpan');
		}
	}

	function loadStatusFromHistory() {
		const history = dbHistory[selectedDate];
		if (history) {
			members.forEach((m, i) => {
				m.status = (history[i] ?? 0) as 0 | 1 | 2;
			});
		} else {
			members.forEach((m) => (m.status = 0));
		}
	}

	function getPreviousDate(dateString: string): string {
		const date = new Date(`${dateString}T12:00:00`);

		date.setDate(date.getDate() - 1);

		return date.toISOString().split('T')[0];
	}

	function getStatusLabel(status: Member['status']): string {
		if (status === 1) return 'TEPAT';
		if (status === 2) return 'TELAT';
		return 'BELUM';
	}

	function getStatusEmoji(status: Member['status']): string {
		if (status === 1) return `${dailyEmoji}⭐`;
		if (status === 2) return dailyEmoji;

		return '–';
	}

	function getStatusReport(status: Member['status']): string {
		if (status === 1) return `TEPAT ${dailyEmoji}⭐`;
		if (status === 2) return `TELAT ${dailyEmoji}`;

		return 'BELUM –';
	}

	function getLastReadingForMember(memberIndex: number): string | null {
		const selected = new Date(`${selectedDate}T12:00:00`);

		// Cari mundur maksimal 365 hari
		for (let i = 1; i <= 365; i++) {
			const date = new Date(selected);

			date.setDate(date.getDate() - i);

			const dateKey = date.toISOString().split('T')[0];

			const history = dbHistory[dateKey];

			if (!history) continue;

			const status = history[memberIndex] ?? 0;

			// Status 1 atau 2 berarti dia pernah melapor
			if (status === 1 || status === 2) {
				return getReadingForDate(dateKey);
			}
		}

		return null;
	}

	function missedYesterday(memberIndex: number): boolean {
		const previousDate = getPreviousDate(selectedDate);

		const previousHistory = dbHistory[previousDate];

		if (!previousHistory) return false;

		return (previousHistory[memberIndex] ?? 0) === 0;
	}

	function toggleStatus(index: number) {
		if (!isCurrentHost) {
			showToast(`Hanya ${hostName} yang dapat mengubah laporan minggu ini`);
			return;
		}

		if (isReadOnly && !godMode) {
			showToast('Pilih ADMIN untuk mengubah riwayat');
			return;
		}

		const member = members[index];
		if (!member) return;

		if (godMode) {
			member.status = ((member.status + 1) % 3) as 0 | 1 | 2;
		} else {
			if (member.status === 0) {
				const nowDate = new Date();
				member.status = (nowDate.getHours() * 60 + nowDate.getMinutes() <= 18 * 60 ? 1 : 2) as
					1 | 2;
			} else {
				member.status = 0;
			}
		}

		saveToDB();
	}

	function toggleGodMode() {
		godMode = !godMode;
		showToast(godMode ? 'Mode admin aktif' : 'Mode admin dikunci');
	}

	function showToast(msg: string) {
		toastMsg = msg;
		toastShow = true;
		clearTimeout(toastTimer);
		toastTimer = setTimeout(() => (toastShow = false), 2000);
	}

	// ========== CLIPBOARD + FALLBACK HTTP ==========
	async function copyText(text: string): Promise<boolean> {
		if (navigator.clipboard?.writeText) {
			try {
				await navigator.clipboard.writeText(text);
				return true;
				// eslint-disable-next-line no-empty
			} catch {}
		}
		const ta = document.createElement('textarea');
		ta.value = text;
		ta.setAttribute('readonly', '');
		ta.style.position = 'fixed';
		ta.style.left = '-9999px';
		document.body.appendChild(ta);
		ta.select();
		const ok = document.execCommand('copy');
		ta.remove();
		return ok;
	}

	// ========== COPY LAPORAN ==========
	async function copyReport() {
		const dateFormatted = new Date(selectedDate)
			.toLocaleDateString('id-ID', {
				weekday: 'long',
				day: 'numeric',
				month: 'long',
				year: 'numeric'
			})
			.toUpperCase();

		const isComplete = !members.some((m) => m.status === 0);
		const timeLabel = isComplete
			? 'LAST UPDATE'
			: 'UPDATE: ' +
				new Date()
					.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })
					.replace('.', ':');

		const memberLines = members
			.map((m, i) => {
				// Jika kemarin tidak melapor
				if (missedYesterday(i)) {
					const lastReading = getLastReadingForMember(i);

					return `${i + 1}. ${m.name} ⛔${lastReading ? ` ${lastReading}` : ''}`;
				}

				// BELUM hari ini
				if (m.status === 0) {
					return `${i + 1}. ${m.name} –`;
				}

				// TEPAT
				if (m.status === 1) {
					return `${i + 1}. ${m.name} ${dailyEmoji}⭐`;
				}

				// TELAT
				return `${i + 1}. ${m.name} ${dailyEmoji}`;
			})
			.join('\n');

		const text =
			`Ⓜ️🅱️🅰️  *MARI BACA ALKITAB GROUP 1 PERJANJIAN LAMA*\n\n` +
			`*MENU BACAAN :*\n` +
			`*${dateFormatted}*\n\n` +
			`📖  *${todayReading}*\n\n` +
			`*${timeLabel}*\n` +
			`${dailyEmoji.repeat(5)}\n\n` +
			`${memberLines}\n\n` +
			`*Firman-Mu itu pelita bagi kakiku dan terang bagi jalanku*\n` +
			`*MAZMUR 119 : 105*`;

		const ok = await copyText(text);
		showToast(ok ? 'Laporan tersalin' : 'Gagal menyalin');
	}

	// ========== COPY MENU ==========
	async function copyTomorrow() {
		if (!previewUnlocked) {
			showToast(`Tunggu sampai ${CONFIG.UNLOCK_HOUR}:00`);
			return;
		}

		const ok = await copyText(tomorrowPreview);
		showToast(ok ? 'Menu besok tersalin' : 'Gagal menyalin');
	}

	// ========== INIT ==========
	$effect(() => {
		const savedTheme = localStorage.getItem(THEME_KEY);
		if (savedTheme === 'dark') darkMode = true;
		else if (savedTheme === 'light') darkMode = false;
		else darkMode = window.matchMedia('(prefers-color-scheme: dark)').matches;
	});
</script>

<svelte:head>
	<title>{title}</title>
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
</svelte:head>

{#if showIdentityConfirm && selectedUser}
	<div class="identity-confirm-overlay">
		<section class="identity-confirm-modal" aria-label="Konfirmasi identitas">
			{#if selectedUserIsHost}
				<!-- ================= HOST CONFIRM ================= -->

				<div class="confirm-warning-icon">⚠</div>

				<div class="confirm-kicker warning">KONFIRMASI AKSES HOST</div>

				<h2>Nama ini sedang mendapat giliran</h2>

				<p class="confirm-description">Kamu memilih:</p>

				<div class="confirm-name host-name">
					{selectedUser}
				</div>

				<div class="host-confirm-info">
					<span>
						{selectedUser} adalah host minggu ini.
					</span>

					<span> Host dapat mengubah status anggota dan menyalin laporan. </span>
				</div>

				<p class="confirm-warning-text">
					Pastikan kamu benar-benar orang yang sedang mendapat giliran host.
				</p>

				<div class="confirm-actions">
					<button type="button" class="confirm-back-btn" onclick={cancelIdentityConfirm}>
						KEMBALI
					</button>

					<button type="button" class="confirm-host-btn" onclick={completeUserLogin}>
						YA, MASUK SEBAGAI {selectedUser.toUpperCase()}
					</button>
				</div>
			{:else}
				<!-- ================= NORMAL USER CONFIRM ================= -->

				<div class="confirm-kicker">KONFIRMASI IDENTITAS</div>

				<h2>Apakah ini nama kamu?</h2>

				<div class="confirm-name">
					{selectedUser}
				</div>

				<p class="confirm-description">
					Kamu akan masuk sebagai
					<strong>{selectedUser}</strong>.
				</p>

				<div class="viewer-confirm-info">Akses kamu: HANYA MELIHAT</div>

				<div class="confirm-actions">
					<button type="button" class="confirm-back-btn" onclick={cancelIdentityConfirm}>
						KEMBALI
					</button>

					<button type="button" class="confirm-enter-btn" onclick={completeUserLogin}>
						YA, BENAR
					</button>
				</div>
			{/if}
		</section>
	</div>
{/if}

<div class="app-shell" class:app-locked={showUserSelector}>
	<!-- ================= HOST FLOAT ================= -->
	<aside class="host-float" aria-label="Informasi giliran melapor">
		<div class="host-float-title">TURN TO REPORT</div>

		<div class="host-float-name">{hostName}</div>

		<div class="host-float-date">{hostWeekText}</div>

		<div class="host-float-note">Host Minggu Ini</div>
	</aside>

	<!-- ================= TOAST ================= -->
	<div id="toast-container">
		<div class="toast" class:show={toastShow} role="status" aria-live="polite">
			<span>{toastMsg}</span>
		</div>
	</div>

	<!-- ================= MAIN DASHBOARD ================= -->
	<main class="dashboard">
		<!-- ================= HEADER ================= -->

		<header class="header">
			<!-- THEME SWITCH -->

			<!-- THEME SWITCH -->
			<div class="theme-wrap">
				<label class="toggle" for="switch">
					<input
						id="switch"
						class="input"
						type="checkbox"
						bind:checked={darkMode}
						aria-label="Ubah tema gelap/terang"
					/>

					<div class="icon icon--moon">
						<svg
							xmlns="http://www.w3.org/2000/svg"
							width="30"
							height="30"
							viewBox="0 0 512 512"
							id="moon"
						>
							<g>
								<path
									fill="#6A6D68"
									d="M412.95,381.15c-8.05,10.119-16.94,19.33-26.55,27.54c-2.271,1.939-4.58,3.819-6.92,5.64   c-0.261,0.21-0.521,0.42-0.78,0.63c-0.09,0.07-0.19,0.13-0.28,0.2c-5.979,4.6-12.2,8.83-18.64,12.689   c-1.92,1.15-3.851,2.28-5.811,3.37c-18.14,10.061-37.819,17.221-58.42,21.16c-12.27,2.34-24.87,3.55-37.66,3.55   c-27.92,0-54.94-5.739-80.32-17.04c-7.74-3.46-15.3-7.43-22.47-11.81c-6.96-4.24-13.77-9-20.24-14.14   c-5.28-4.19-10.3-8.62-15.07-13.25c-1.3-1.261-2.57-2.54-3.82-3.83c-30.43-31.21-49.57-71.37-54.6-115.38   c-4.54-39.75,2.83-79.04,20.95-113.75c4.99-9.561,10.81-18.78,17.41-27.561c0.2-0.26,0.4-0.529,0.6-0.79   c0.9-1.18,1.81-2.359,2.74-3.529c37.77-47.521,94.29-74.78,155.07-74.78c45.101,0,87.641,14.87,123.021,42.99   c1.54,1.22,2.89,2.33,4.14,3.39c3.16,2.64,6.29,5.43,9.51,8.5c0.49,0.47,0.99,0.94,1.471,1.43c1.3,1.25,2.58,2.54,3.84,3.83   c32.41,33.351,51.979,77.011,55.31,123.75C458.97,293.51,443.88,342.23,412.95,381.15z"
									opacity=".9"
								></path>
								<path
									fill="#A3AAA0"
									d="M408.95,377.15c-8.05,10.119-16.94,19.33-26.55,27.54c-2.271,1.939-4.58,3.819-6.92,5.64   c-0.261,0.21-0.521,0.42-0.78,0.63c-0.09,0.07-0.19,0.13-0.28,0.2c-5.979,4.6-12.2,8.83-18.64,12.689   c-1.92,1.15-3.851,2.28-5.811,3.37c-19.76,10.96-41.359,18.471-63.979,22.141c-10.51,1.699-21.23,2.569-32.101,2.569   c-27.92,0-54.94-5.739-80.32-17.04c-7.74-3.46-15.3-7.43-22.47-11.81c-6.96-4.24-13.77-9-20.24-14.14   c-5.21-4.141-10.17-8.511-14.89-13.08c-0.06-0.051-0.12-0.11-0.18-0.17c-32.64-31.721-53.18-73.381-58.42-119.21   c-4.54-39.75,2.83-79.04,20.95-113.75c4.99-9.561,10.81-18.78,17.41-27.561c1.09-1.449,2.2-2.89,3.34-4.319   c0.55-0.69,1.1-1.37,1.65-2.051c37.76-46.25,93.52-72.729,153.42-72.729c45.101,0,87.641,14.87,123.021,42.99   c1.54,1.22,2.89,2.33,4.14,3.39c3.16,2.64,6.29,5.43,9.51,8.5c1.811,1.72,3.58,3.48,5.311,5.26c0.05,0.061,0.11,0.11,0.16,0.17   c32.319,33.33,51.83,76.92,55.149,123.58C454.97,289.51,439.88,338.23,408.95,377.15z"
								></path>
								<circle
									cx="285"
									cy="156"
									r="44.5"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="385"
									cy="300"
									r="21.5"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="166"
									cy="296.5"
									r="27.84"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="261.25"
									cy="272.75"
									r="14.75"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="151.5"
									cy="184"
									r="28"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="297.5"
									cy="382.501"
									r="27.5"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="395"
									cy="213"
									r="18.5"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<circle
									cx="317"
									cy="216"
									r="8"
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
								></circle>
								<path
									fill="#666865"
									stroke="#5E5E5D"
									stroke-miterlimit="10"
									stroke-width="4"
									d="M299.55,450.38   c-12.27,2.34-24.87,3.55-37.66,3.55c-27.92,0-54.94-5.739-80.32-17.04c-7.74-3.46-15.3-7.43-22.47-11.81   c-6.96-4.24-13.77-9-20.24-14.14c-5.28-4.19-10.3-8.62-15.07-13.25c-1.3-1.261-2.57-2.54-3.82-3.83   c-0.06-0.051-0.12-0.11-0.18-0.17c-32.64-31.721-53.18-73.381-58.42-119.21c-4.54-39.75,2.83-79.04,20.95-113.75   c4.99-9.561,10.81-18.78,17.41-27.561c1.09-1.449,2.2-2.89,3.34-4.319c0.55-0.69,1.1-1.37,1.65-2.051   c-0.16,3.011-0.29,6.2-0.39,9.58c-2.39,79.15,12.97,253.43,185.661,310.98C293.12,448.41,296.31,449.42,299.55,450.38z"
									opacity=".2"
								></path>
							</g>
						</svg>
					</div>

					<div class="w-8 icon icon--sun">
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 47.5 47.5" id="sun">
							<defs>
								<clipPath id="a">
									<path d="M0 38h38V0H0v38Z"></path>
								</clipPath>
							</defs>
							<g fill="#ffac33" clip-path="url(#a)" transform="matrix(1.25 0 0 -1.25 0 47.5)">
								<path
									d="M17 35s0 2 2 2 2-2 2-2v-2s0-2-2-2-2 2-2 2v2zM35 21s2 0 2-2-2-2-2-2h-2s-2 0-2 2 2 2 2 2h2zM5 21s2 0 2-2-2-2-2-2H3s-2 0-2 2 2 2 2 2h2zM10.121 29.706s1.414-1.414 0-2.828-2.828 0-2.828 0l-1.415 1.414s-1.414 1.414 0 2.829c1.415 1.414 2.829 0 2.829 0l1.414-1.415ZM31.121 8.707s1.414-1.414 0-2.828-2.828 0-2.828 0l-1.414 1.414s-1.414 1.414 0 2.828 2.828 0 2.828 0l1.414-1.414ZM30.708 26.879s-1.414-1.414-2.828 0 0 2.828 0 2.828l1.414 1.414s1.414 1.414 2.828 0 0-2.828 0-2.828l-1.414-1.414ZM9.708 5.879s-1.414-1.414-2.828 0 0 2.828 0 2.828l1.414 1.414s1.414 1.414 2.828 0 0-2.828 0-2.828L9.708 5.879ZM17 5s0 2 2 2 2-2 2-2V3s0-2-2-2-2 2-2 2v2zM29 19c0 5.523-4.478 10-10 10-5.523 0-10-4.477-10-10 0-5.522 4.477-10 10-10 5.522 0 10 4.478 10 10"
								></path>
							</g>
						</svg>
					</div>
				</label>
			</div>

			<!-- BRAND -->

			<div class="brand">
				<div class="brand-title">MARI BACA ALKITAB</div>

				<div class="brand-subtitle">GROUP 1</div>
			</div>

			<!-- DATE -->

			<div class="date-header">{dateText}</div>

			<!-- CLOCK -->

			<div class="clock-huge">{clockText}</div>

			<!-- DATABASE -->

			<div class="db-badge" data-state={dbStatus}>
				{dbStatus === 'online'
					? 'TURSO · ONLINE'
					: dbStatus === 'saving'
						? 'TURSO · MENYIMPAN...'
						: dbStatus === 'error'
							? 'TURSO · ERROR'
							: 'CONNECTING TO TURSO DB...'}
			</div>

			<div class="user-session">
				<div class="current-user">
					<span class="current-user-label"> LOGIN AS: </span>

					<div class="current-user-info">
						<strong>{currentUser}</strong>

						{#if isCurrentHost}
							<span class="access-badge host-access"> HOST • BISA EDIT </span>
						{:else}
							<span class="access-badge viewer-access"> READ ONLY </span>
						{/if}
					</div>
				</div>

				<button class="logout-btn" type="button" onclick={logoutUser}> LOGOUT </button>
			</div>
		</header>

		<!-- ================= CONTROLS ================= -->

		<section class="controls">
			<div class="input-group">
				<div class="section-label">REPORT DATE</div>

				<div class="date-wrapper">
					<input type="date" aria-label="Tanggal laporan" bind:value={selectedDate} />

					<button
						class="btn-admin"
						class:on={godMode}
						type="button"
						onclick={toggleGodMode}
						aria-pressed={godMode}
						title={godMode ? 'Mode admin ON — klik untuk OFF' : 'Mode admin OFF — klik untuk ON'}
					>
						<span class="back"></span>
						<span class="front"></span>
					</button>
				</div>

				<div class={bannerClass}>{bannerText}</div>
			</div>

			<div class="scripture-box">
				<div class="scripture-label">BACAAN HARI INI</div>
				<div class="bacaan-text">{todayReading}</div>
			</div>
		</section>

		<!-- ================= MEMBER SECTION ================= -->

		<section class="member-section">
			<!-- HEADING -->

			<div class="list-heading">
				<div>
					<div class="list-title">MEMBER REPORT</div>

					<div class="list-subtitle">Tekan nama untuk mengubah status bacaan.</div>
				</div>

				<!-- LEGEND -->

				<div class="status-legend" aria-label="Keterangan status">
					<span>
						<i class="legend-dot pending"></i>
						Belum
					</span>

					<span>
						<i class="legend-dot ontime"></i>
						Tepat
					</span>

					<span>
						<i class="legend-dot late"></i>
						Telat
					</span>
				</div>
			</div>

			<!-- MEMBER LIST -->

			<div class="list-container" class:list-locked={isReadOnly && !godMode}>
				{#each members as member, index (member.name)}
					<button
						class="member-item st-{member.status}"
						class:locked={!isCurrentHost}
						type="button"
						disabled={!isCurrentHost}
						onclick={() => toggleStatus(index)}
					>
						<div class="num-col">{pad(index + 1)}</div>
						<div class="name-wrapper">
							<span class="name-col">{member.name}</span>
						</div>
						<div class="badge">
							{getStatusLabel(member.status)}
						</div>
					</button>
				{/each}
			</div>
		</section>

		<!-- ================= FOOTER ================= -->

		<!-- ================= FOOTER ================= -->

		<footer class="footer-main">
			<!-- COPY LAPORAN -->
			{#if isCurrentHost}
				<button class="btn-report" type="button" onclick={copyReport}> COPY LAPORAN </button>
			{/if}

			<!-- PREVIEW BESOK -->
			<div class="future-section">
				<div class="future-tag">PREVIEW BESOK</div>

				<div class="preview-note">Menu besok dapat disalin mulai pukul 19:00.</div>

				<div id="previewTomorrow">
					{tomorrowPreview}
				</div>

				{#if isCurrentHost}
					<button
						class="btn-future"
						class:btn-active-purple={previewUnlocked}
						class:btn-is-locked={!previewUnlocked}
						type="button"
						onclick={copyTomorrow}
					>
						{previewUnlocked ? 'COPY MENU BACAAN' : `HARAP TUNGGU ${CONFIG.UNLOCK_HOUR}:00`}
					</button>
				{/if}
			</div>
		</footer>
	</main>
</div>
{#if showUserSelector}
	<div class="user-selector-overlay">
		<section class="user-selector-modal" aria-label="Pilih pengguna">
			<div class="user-selector-header">
				<div class="user-selector-kicker">MARI BACA ALKITAB</div>

				<h1>Siapa kamu?</h1>

				<p>Pilih nama kamu untuk masuk ke laporan Mari Baca Alkitab Group 1.</p>
			</div>

			<div class="user-list">
				{#each members as member, index (member.name)}
					<button
						type="button"
						class="user-option"
						class:selected={selectedUser === member.name}
						onclick={() => (selectedUser = member.name)}
					>
						<span class="user-number">
							{pad(index + 1)}
						</span>

						<span class="user-name">
							{member.name}
						</span>

						<span class="user-check">
							{selectedUser === member.name ? '✓' : ''}
						</span>
					</button>
				{/each}
			</div>

			<button
				type="button"
				class="confirm-user-btn"
				class:ready={selectedUser !== null}
				disabled={selectedUser === null}
				onclick={confirmUser}
			>
				{selectedUser ? `MASUK SEBAGAI ${selectedUser.toUpperCase()}` : 'PILIH NAMA'}
			</button>
		</section>
	</div>
{/if}
