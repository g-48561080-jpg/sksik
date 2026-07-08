import React, { useState, useEffect, useMemo } from 'react';
import { initializeApp } from 'firebase/app';
import { 
  getAuth, 
  signInWithCustomToken, 
  signInAnonymously, 
  onAuthStateChanged,
  signOut
} from 'firebase/auth';
import { 
  getFirestore, 
  collection, 
  doc, 
  setDoc, 
  addDoc, 
  getDocs, 
  updateDoc, 
  deleteDoc, 
  onSnapshot, 
  query, 
  where 
} from 'firebase/firestore';

// Lucide Icons (Inline SVGs for reliable render)
const Icons = {
  Calendar: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z" /></svg>,
  Users: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" /></svg>,
  Home: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" /></svg>,
  Plus: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 4v16m8-8H4" /></svg>,
  Trash: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>,
  Check: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M5 13l4 4L19 7" /></svg>,
  X: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M6 18L18 6M6 6l12 12" /></svg>,
  Bell: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" /></svg>,
  LogOut: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" /></svg>,
  Sun: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364-6.364l-.707.707M6.343 17.657l-.707.707m12.728 0l-.707-.707M6.343 6.343l-.707-.707M12 7a5 5 0 100 10 5 5 0 000-10z" /></svg>,
  Moon: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" /></svg>,
  Printer: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M17 17h2a2 2 0 002-2v-4a2 2 0 00-2-2H5a2 2 0 00-2 2v4a2 2 0 002 2h2m2 4h6a2 2 0 002-2v-4a2 2 0 00-2-2H9a2 2 0 00-2 2v4a2 2 0 002 2zm8-12V5a2 2 0 00-2-2H9a2 2 0 00-2 2v4h10z" /></svg>,
  FileText: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" /></svg>,
  Search: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>,
  Info: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>,
  Shield: () => <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z" /></svg>,
  Menu: () => <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M4 6h16M4 12h16M4 18h16" /></svg>
};

// Generasi Slot Masa (07:30 hingga 16:30 / 4:30 petang, selang 30 minit)
const generateTimeSlots = () => {
  const slots = [];
  let currentHour = 7;
  let currentMin = 30;
  // 4:30 petang bersamaan jam 16:30
  while (currentHour < 16 || (currentHour === 16 && currentMin === 30)) {
    const formattedHour = currentHour.toString().padStart(2, '0');
    const formattedMin = currentMin.toString().padStart(2, '0');
    slots.push(`${formattedHour}:${formattedMin}`);
    currentMin += 30;
    if (currentMin >= 60) {
      currentMin = 0;
      currentHour += 1;
    }
  }
  return slots;
};

const TIME_SLOTS = generateTimeSlots();

// Senarai Bilik Khas Asal
const DEFAULT_ROOMS = [
  { id: 'room-1', nama: 'Makmal Komputer', lokasi: 'Blok A, Aras 2', status: 'Aktif' },
  { id: 'room-2', nama: 'Bilik Mesyuarat', lokasi: 'Blok Pentadbiran, Aras G', status: 'Aktif' },
  { id: 'room-3', nama: 'Bilik Akses', lokasi: 'Pusat Sumber, Aras 1', status: 'Aktif' },
  { id: 'room-4', nama: 'Studio Digital', lokasi: 'Blok B, Aras 3', status: 'Aktif' },
  { id: 'room-5', nama: 'Pusat Sumber Sekolah', lokasi: 'Blok Utama, Aras 1', status: 'Aktif' }
];

// Firebase Global Config Fallbacks & Init
const firebaseConfig = typeof __firebase_config !== 'undefined' 
  ? JSON.parse(__firebase_config) 
  : { apiKey: "", authDomain: "mock-stbks.firebaseapp.com", projectId: "mock-stbks", storageBucket: "mock-stbks.appspot.com" };

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const appId = typeof __app_id !== 'undefined' ? __app_id : 'stbks-app';

export default function App() {
  // Global & Authentication States
  const [user, setUser] = useState(null);
  const [userRole, setUserRole] = useState('Guru'); // 'Guru' atau 'Admin'
  const [isDarkMode, setIsDarkMode] = useState(false);
  const [sidebarOpen, setSidebarOpen] = useState(false);
  const [notifications, setNotifications] = useState([
    { id: 1, mesej: "Sistem Tempahan Bilik Khas Sekolah sedia digunakan.", masa: "Baru sahaja", dibaca: false }
  ]);
  const [showNotifPanel, setShowNotifPanel] = useState(false);

  // Data States
  const [rooms, setRooms] = useState(DEFAULT_ROOMS);
  const [bookings, setBookings] = useState([]);
  const [auditLogs, setAuditLogs] = useState([]);

  // Form & View States
  const [currentTab, setCurrentTab] = useState('dashboard'); // 'dashboard', 'booking-form', 'calendar', 'manage-rooms', 'reports', 'audit'
  const [searchQuery, setSearchQuery] = useState('');
  const [filterStatus, setFilterStatus] = useState('Semua');
  const [filterRoom, setFilterRoom] = useState('Semua');

  // Booking Form State
  const [formData, setFormData] = useState({
    teacherId: '',
    teacherName: '',
    teacherPhone: '',
    teacherEmail: '',
    jawatan: 'Guru Akademik Biasa',
    roomId: 'room-1',
    tarikh: new Date().toISOString().split('T')[0],
    hari: getMalayDayName(new Date().toISOString().split('T')[0]),
    masaMula: '08:00',
    masaTamat: '10:00',
    bilMurid: 30,
    tujuan: ''
  });

  // Modal / Feedback State
  const [modalFeedback, setModalFeedback] = useState(null); // { type: 'success'|'error'|'conflict', message: '' }
  const [selectedBookingDetails, setSelectedBookingDetails] = useState(null); // Untuk paparan Slip / Cetakan
  const [showSlipModal, setShowSlipModal] = useState(false);
  
  // Custom Firebase Auth Initialization
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Authentication Error", error);
      }
    };
    initAuth();

    const unsubscribe = onAuthStateChanged(auth, (firebaseUser) => {
      if (firebaseUser) {
        setUser(firebaseUser);
        // Daftarkan maklumat asas guru secara automatik
        setFormData(prev => ({
          ...prev,
          teacherId: firebaseUser.uid,
          teacherEmail: firebaseUser.email || `${firebaseUser.uid}@moe-dl.edu.my`,
          teacherName: firebaseUser.displayName || 'Guru DELIMa'
        }));
      } else {
        setUser(null);
      }
    });

    return () => unsubscribe();
  }, []);

  // Syncing Data from Firestore (With In-Memory Fallback if Firebase not ready)
  useEffect(() => {
    if (!user) return;

    // Realtime Bookings Listener
    const bookingsCol = collection(db, 'artifacts', appId, 'public', 'data', 'bookings');
    const unsubBookings = onSnapshot(bookingsCol, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setBookings(data);
    }, (err) => {
      console.warn("Firestore listener failed, using in-memory demo data.", err);
      // Fallback demo bookings
      setBookings([
        {
          id: 'b-1',
          roomId: 'room-1',
          teacherId: 'user-demo',
          teacherName: 'Cikgu Ahmad Fauzi',
          teacherPhone: '0123456789',
          teacherEmail: 'ahmad@moe-dl.edu.my',
          jawatan: 'Ketua Panitia',
          tarikh: new Date().toISOString().split('T')[0],
          hari: getMalayDayName(new Date().toISOString().split('T')[0]),
          masaMula: '08:00',
          masaTamat: '09:30',
          bilMurid: 35,
          tujuan: 'Sesi Amali Sains Komputer Bab 3',
          status: 'Diluluskan',
          createdAt: new Date().toISOString()
        },
        {
          id: 'b-2',
          roomId: 'room-2',
          teacherId: 'user-another',
          teacherName: 'Cikgu Noraini binti Zakaria',
          teacherPhone: '0198765432',
          teacherEmail: 'noraini@moe-dl.edu.my',
          jawatan: 'Guru Akademik',
          tarikh: new Date().toISOString().split('T')[0],
          hari: getMalayDayName(new Date().toISOString().split('T')[0]),
          masaMula: '10:00',
          masaTamat: '12:00',
          bilMurid: 15,
          tujuan: 'Mesyuarat Guru Akademik Bil. 2',
          status: 'Menunggu',
          createdAt: new Date().toISOString()
        }
      ]);
    });

    // Realtime Rooms Listener
    const roomsCol = collection(db, 'artifacts', appId, 'public', 'data', 'rooms');
    const unsubRooms = onSnapshot(roomsCol, (snapshot) => {
      if (!snapshot.empty) {
        const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        setRooms(data);
      } else {
        // Init Firestore with default rooms if empty
        DEFAULT_ROOMS.forEach(async (r) => {
          await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'rooms', r.id), r);
        });
      }
    }, (err) => {
      console.warn("Firestore rooms listener failed, using defaults.");
    });

    // Realtime Logs Listener
    const logsCol = collection(db, 'artifacts', appId, 'public', 'data', 'logs');
    const unsubLogs = onSnapshot(logsCol, (snapshot) => {
      const data = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
      setAuditLogs(data.sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp)));
    }, (err) => {
      setAuditLogs([
        { id: 'l-1', pengguna: 'Sistem', tindakan: 'Sistem diaktifkan', timestamp: new Date().toLocaleString() }
      ]);
    });

    return () => {
      unsubBookings();
      unsubRooms();
      unsubLogs();
    };
  }, [user]);

  // Fungsi Pembantu Hari dalam Bahasa Melayu
  function getMalayDayName(dateString) {
    const days = ['Ahad', 'Isnin', 'Selasa', 'Rabu', 'Khamis', 'Jumaat', 'Sabtu'];
    const date = new Date(dateString);
    return days[date.getDay()];
  }

  // Update Hari apabila Tarikh berubah dalam Form
  useEffect(() => {
    setFormData(prev => ({
      ...prev,
      hari: getMalayDayName(prev.tarikh)
    }));
  }, [formData.tarikh]);

  // Log Audit Helper
  const writeLog = async (tindakan) => {
    const logData = {
      pengguna: userRole === 'Admin' ? 'Admin (Sistem)' : (formData.teacherName || 'Guru'),
      tindakan,
      timestamp: new Date().toLocaleString()
    };
    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'logs'), logData);
    } catch (e) {
      setAuditLogs(prev => [ { id: Date.now().toString(), ...logData }, ...prev ]);
    }
  };

  // Notifikasi Generator
  const addNotification = (mesej) => {
    const newNotif = {
      id: Date.now(),
      mesej,
      masa: "Baru sahaja",
      dibaca: false
    };
    setNotifications(prev => [newNotif, ...prev]);
  };

  // Formula Konflik Utama
  // Mengembalikan true jika ada konflik, false jika tiada bertindih
  const semakPertindihan = (targetRoomId, targetTarikh, targetMasaMula, targetMasaTamat, excludeBookingId = null) => {
    const formatMinutes = (timeStr) => {
      const [h, m] = timeStr.split(':').map(Number);
      return h * 60 + m;
    };

    const targetMulaMin = formatMinutes(targetMasaMula);
    const targetTamatMin = formatMinutes(targetMasaTamat);

    // Cari tempahan bilik yang sama pada tarikh yang sama dan bukan status Ditolak
    const matchedBookings = bookings.filter(b => 
      b.roomId === targetRoomId && 
      b.tarikh === targetTarikh && 
      b.status !== 'Ditolak' &&
      b.id !== excludeBookingId
    );

    for (const b of matchedBookings) {
      const bMulaMin = formatMinutes(b.masaMula);
      const bTamatMin = formatMinutes(b.masaTamat);

      // Formula Konflik: masaMulaBaharu < masaTamatSediaAda && masaTamatBaharu > masaMulaSediaAda
      if (targetMulaMin < bTamatMin && targetTamatMin > bMulaMin) {
        return b; // Mengembalikan maklumat tempahan yang bertindih
      }
    }
    return null;
  };

  // Live Konflik Checker semasa isi borang
  const liveConflict = useMemo(() => {
    if (!formData.roomId || !formData.tarikh || !formData.masaMula || !formData.masaTamat) return null;
    return semakPertindihan(formData.roomId, formData.tarikh, formData.masaMula, formData.masaTamat);
  }, [formData.roomId, formData.tarikh, formData.masaMula, formData.masaTamat, bookings]);

  // Hantar Tempahan
  const handleHantarTempahan = async (e) => {
    e.preventDefault();

    // Semakan hari bekerja (Ahad hingga Khamis)
    const hariPilihan = getMalayDayName(formData.tarikh);
    if (hariPilihan === 'Jumaat' || hariPilihan === 'Sabtu') {
      setModalFeedback({
        type: 'error',
        message: 'Maaf. Sistem hanya membenarkan tempahan pada hari bekerja (Ahad hingga Khamis) sahaja.'
      });
      return;
    }

    // Pastikan tiada pertembungan saat akhir
    const konflik = semakPertindihan(formData.roomId, formData.tarikh, formData.masaMula, formData.masaTamat);
    if (konflik) {
      setModalFeedback({
        type: 'conflict',
        message: `Maaf. Slot ini telah ditempah oleh ${konflik.teacherName} (${konflik.masaMula} - ${konflik.masaTamat}). Sila pilih masa atau tarikh lain.`
      });
      return;
    }

    const dataBaru = {
      ...formData,
      teacherId: user?.uid || 'guest-user',
      status: 'Menunggu',
      createdAt: new Date().toISOString()
    };

    try {
      await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'bookings'), dataBaru);
      writeLog(`Membuat tempahan baharu di ${rooms.find(r => r.id === formData.roomId)?.nama} pada ${formData.tarikh}`);
      addNotification(`Tempahan baharu bagi ${rooms.find(r => r.id === formData.roomId)?.nama} berjaya dihantar.`);
      
      setModalFeedback({
        type: 'success',
        message: 'Tempahan berjaya dihantar! Notifikasi emel simulasi telah dihantar kepada pentadbir sistem untuk kelulusan.'
      });

      // Reset Form kecuali maklumat peribadi
      setFormData(prev => ({
        ...prev,
        tujuan: '',
        bilMurid: 30
      }));
    } catch (err) {
      console.error(err);
      // Fallback update in local state if firestore offline
      const localId = 'b-' + Date.now();
      setBookings(prev => [...prev, { id: localId, ...dataBaru }]);
      setModalFeedback({
        type: 'success',
        message: 'Tempahan berjaya dihantar! (Sesi Tempatan)'
      });
    }
  };

  // Pengurusan Tempahan oleh Admin / Guru Sendiri (Delete)
  const handleBatalTempahan = async (id) => {
    const bookingToCancel = bookings.find(b => b.id === id);
    if (!bookingToCancel) return;

    if (confirm("Adakah anda pasti mahu membatalkan tempahan ini?")) {
      try {
        await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'bookings', id));
        writeLog(`Membatalkan tempahan ID: ${id}`);
        addNotification(`Tempahan pada ${bookingToCancel.tarikh} telah dibatalkan.`);
      } catch (err) {
        setBookings(prev => prev.filter(b => b.id !== id));
      }
    }
  };

  // Kemas Kini Status Tempahan (Admin Sahaja)
  const handleTukarStatus = async (id, statusBaharu) => {
    try {
      const bookingRef = doc(db, 'artifacts', appId, 'public', 'data', 'bookings', id);
      await updateDoc(bookingRef, { status: statusBaharu });
      
      const booking = bookings.find(b => b.id === id);
      writeLog(`Mengubah status tempahan ID: ${id} kepada ${statusBaharu}`);
      addNotification(`Tempahan bilik ${rooms.find(r => r.id === booking?.roomId)?.nama} telah ${statusBaharu}.`);
    } catch (err) {
      setBookings(prev => prev.map(b => b.id === id ? { ...b, status: statusBaharu } : b));
    }
  };

  // Pengurusan Bilik (Admin)
  const [newRoomName, setNewRoomName] = useState('');
  const [newRoomLoc, setNewRoomLoc] = useState('');

  const handleTambahBilik = async (e) => {
    e.preventDefault();
    if (!newRoomName || !newRoomLoc) return;

    const rId = 'room-' + Date.now();
    const rBaru = { id: rId, nama: newRoomName, lokasi: newRoomLoc, status: 'Aktif' };

    try {
      await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'rooms', rId), rBaru);
      writeLog(`Menambah bilik khas baharu: ${newRoomName}`);
      setNewRoomName('');
      setNewRoomLoc('');
    } catch (err) {
      setRooms(prev => [...prev, rBaru]);
    }
  };

  const handlePadamBilik = async (id) => {
    if (confirm("Padam bilik ini? Semua tempahan bilik ini mungkin terjejas.")) {
      try {
        await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'rooms', id));
        writeLog(`Memadam bilik khas ID: ${id}`);
      } catch (err) {
        setRooms(prev => prev.filter(r => r.id !== id));
      }
    }
  };

  // Filter & Search Logic (Carian disesuaikan tanpa Panitia & Kelas)
  const filteredBookings = useMemo(() => {
    return bookings.filter(b => {
      const matchedQuery = 
        b.teacherName.toLowerCase().includes(searchQuery.toLowerCase()) ||
        b.tujuan.toLowerCase().includes(searchQuery.toLowerCase()) ||
        (rooms.find(r => r.id === b.roomId)?.nama || '').toLowerCase().includes(searchQuery.toLowerCase());
      
      const matchedStatus = filterStatus === 'Semua' || b.status === filterStatus;
      const matchedRoom = filterRoom === 'Semua' || b.roomId === filterRoom;

      return matchedQuery && matchedStatus && matchedRoom;
    });
  }, [bookings, searchQuery, filterStatus, filterRoom, rooms]);

  // Statistik untuk Laporan / Dashboard
  const stats = useMemo(() => {
    const total = bookings.length;
    const diluluskan = bookings.filter(b => b.status === 'Diluluskan').length;
    const menunggu = bookings.filter(b => b.status === 'Menunggu').length;
    const ditolak = bookings.filter(b => b.status === 'Ditolak').length;

    // Statistik bulanan untuk carta simulasi
    const bulanCounts = {};
    bookings.forEach(b => {
      const bulanStr = new Date(b.tarikh).toLocaleString('ms-MY', { month: 'long' });
      bulanCounts[bulanStr] = (bulanCounts[bulanStr] || 0) + 1;
    });

    // Statistik penggunaan bilik
    const bilikCounts = {};
    bookings.forEach(b => {
      const roomName = rooms.find(r => r.id === b.roomId)?.nama || 'Tidak Diketahui';
      bilikCounts[roomName] = (bilikCounts[roomName] || 0) + 1;
    });

    return { total, diluluskan, menunggu, ditolak, bulanCounts, bilikCounts };
  }, [bookings, rooms]);

  // Simulasi Muat Turun CSV / Excel
  const handleExportExcel = () => {
    let csvContent = "data:text/csv;charset=utf-8,";
    csvContent += "ID Tempahan,Guru,Jawatan,Bilik,Tarikh,Masa,Status,Tujuan\n";
    
    filteredBookings.forEach(b => {
      const roomName = rooms.find(r => r.id === b.roomId)?.nama || '';
      csvContent += `"${b.id}","${b.teacherName}","${b.jawatan}","${roomName}","${b.tarikh}","${b.masaMula} - ${b.masaTamat}","${b.status}","${b.tujuan}"\n`;
    });

    const encodedUri = encodeURI(csvContent);
    const link = document.createElement("a");
    link.setAttribute("href", encodedUri);
    link.setAttribute("download", `Laporan_Tempahan_STBKS_${new Date().toISOString().split('T')[0]}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    writeLog("Mengeksport data tempahan ke CSV/Excel");
  };

  // Cetakan Halaman Laporan
  const handleCetakLaporan = () => {
    window.print();
    writeLog("Melakukan cetakan laporan sistem");
  };

  return (
    <div className={`min-h-screen transition-colors duration-300 ${isDarkMode ? 'bg-[#0F172A] text-slate-100' : 'bg-[#F8FAFC] text-slate-800'}`}>
      
      {/* HEADER UTAMA */}
      <header className={`sticky top-0 z-40 backdrop-blur-md border-b transition-colors duration-300 ${isDarkMode ? 'bg-[#1E293B]/80 border-slate-700' : 'bg-white/80 border-slate-200'} shadow-sm`}>
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
          <div className="flex items-center space-x-3">
            <button 
              onClick={() => setSidebarOpen(!sidebarOpen)}
              className="lg:hidden p-2 rounded-lg hover:bg-slate-200 dark:hover:bg-slate-700 transition"
            >
              <Icons.Menu />
            </button>
            <div className="bg-gradient-to-tr from-blue-600 to-emerald-500 p-2.5 rounded-xl shadow-md text-white">
              <Icons.Calendar />
            </div>
            <div>
              <h1 className="text-lg font-bold tracking-tight bg-gradient-to-r from-blue-600 to-emerald-500 bg-clip-text text-transparent">STBKS</h1>
              <p className="text-xs text-slate-500 dark:text-slate-400 font-medium">Sistem Tempahan Bilik Khas Sekolah</p>
            </div>
          </div>

          {/* Quick Controls & User Profile */}
          <div className="flex items-center space-x-3">
            {/* Dark Mode Switcher */}
            <button 
              onClick={() => setIsDarkMode(!isDarkMode)} 
              className={`p-2 rounded-xl transition ${isDarkMode ? 'bg-slate-800 text-yellow-400 hover:bg-slate-700' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}`}
              title="Tukar Tema"
            >
              {isDarkMode ? <Icons.Sun /> : <Icons.Moon />}
            </button>

            {/* Notification Center */}
            <div className="relative">
              <button 
                onClick={() => setShowNotifPanel(!showNotifPanel)}
                className={`p-2 rounded-xl relative transition ${isDarkMode ? 'bg-slate-800 text-slate-300 hover:bg-slate-700' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}`}
              >
                <Icons.Bell />
                {notifications.some(n => !n.dibaca) && (
                  <span className="absolute top-1.5 right-1.5 w-2.5 h-2.5 bg-red-500 rounded-full animate-pulse"></span>
                )}
              </button>
              
              {showNotifPanel && (
                <div className={`absolute right-0 mt-2 w-80 rounded-2xl border shadow-xl p-4 transition-all duration-200 z-50 ${isDarkMode ? 'bg-[#1E293B] border-slate-700' : 'bg-white border-slate-100'}`}>
                  <div className="flex justify-between items-center mb-3">
                    <h3 className="font-bold text-sm">Notifikasi Terkini</h3>
                    <button 
                      onClick={() => setNotifications(prev => prev.map(n => ({...n, dibaca: true})))} 
                      className="text-xs text-blue-500 hover:underline"
                    >
                      Tanda semua dibaca
                    </button>
                  </div>
                  <div className="space-y-2.5 max-h-60 overflow-y-auto">
                    {notifications.map(notif => (
                      <div key={notif.id} className={`p-2.5 rounded-xl text-xs transition-colors ${notif.dibaca ? 'opacity-70' : 'bg-blue-50/50 dark:bg-blue-900/20'}`}>
                        <p className="font-medium text-slate-700 dark:text-slate-200">{notif.mesej}</p>
                        <span className="text-[10px] text-slate-400 block mt-1">{notif.masa}</span>
                      </div>
                    ))}
                  </div>
                </div>
              )}
            </div>

            {/* Role Switcher */}
            <div className={`hidden md:flex items-center space-x-1 p-1 rounded-xl border ${isDarkMode ? 'bg-slate-800/50 border-slate-700' : 'bg-slate-100 border-slate-200'}`}>
              <button 
                onClick={() => { setUserRole('Guru'); writeLog("Tukar peranan ke Guru"); }}
                className={`px-3 py-1 text-xs font-semibold rounded-lg transition-all ${userRole === 'Guru' ? 'bg-blue-600 text-white shadow-sm' : 'text-slate-500 hover:text-slate-800 dark:hover:text-slate-200'}`}
              >
                Mod Guru
              </button>
              <button 
                onClick={() => { setUserRole('Admin'); writeLog("Tukar peranan ke Admin"); }}
                className={`px-3 py-1 text-xs font-semibold rounded-lg transition-all ${userRole === 'Admin' ? 'bg-emerald-600 text-white shadow-sm' : 'text-slate-500 hover:text-slate-800 dark:hover:text-slate-200'}`}
              >
                Mod Admin
              </button>
            </div>

            {/* Profile Avatar */}
            <div className="flex items-center space-x-2 pl-2 border-l border-slate-200 dark:border-slate-700">
              <div className="w-9 h-9 rounded-full bg-gradient-to-tr from-blue-600 to-indigo-600 flex items-center justify-center text-white font-bold text-sm">
                {userRole === 'Admin' ? 'AD' : 'CG'}
              </div>
              <div className="hidden lg:block text-left">
                <p className="text-xs font-bold leading-none">{userRole === 'Admin' ? 'Pentadbir Utama' : 'Cikgu Delima'}</p>
                <span className="text-[10px] text-slate-400 font-medium">{userRole === 'Admin' ? 'Admin Sistem' : 'Guru Akademik'}</span>
              </div>
            </div>
          </div>
        </div>
      </header>

      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <div className="lg:grid lg:grid-cols-12 lg:gap-8">
          
          {/* SIDEBAR NAVIGATION */}
          <aside className={`lg:col-span-3 mb-6 lg:mb-0 ${sidebarOpen ? 'block' : 'hidden lg:block'}`}>
            <nav className="space-y-1.5 sticky top-24">
              <button 
                onClick={() => { setCurrentTab('dashboard'); setSidebarOpen(false); }}
                className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl font-medium text-sm transition-all ${currentTab === 'dashboard' ? 'bg-blue-600 text-white shadow-lg shadow-blue-500/15' : 'hover:bg-slate-150 dark:hover:bg-slate-800'}`}
              >
                <Icons.Home />
                <span>Dashboard Utama</span>
              </button>

              <button 
                onClick={() => { setCurrentTab('booking-form'); setSidebarOpen(false); }}
                className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl font-medium text-sm transition-all ${currentTab === 'booking-form' ? 'bg-blue-600 text-white shadow-lg shadow-blue-500/15' : 'hover:bg-slate-150 dark:hover:bg-slate-800'}`}
              >
                <Icons.Plus />
                <span>Borang Tempahan Baru</span>
              </button>

              <button 
                onClick={() => { setCurrentTab('calendar'); setSidebarOpen(false); }}
                className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl font-medium text-sm transition-all ${currentTab === 'calendar' ? 'bg-blue-600 text-white shadow-lg shadow-blue-500/15' : 'hover:bg-slate-150 dark:hover:bg-slate-800'}`}
              >
                <Icons.Calendar />
                <span>Kalendar Slot Masa</span>
              </button>

              <button 
                onClick={() => { setCurrentTab('reports'); setSidebarOpen(false); }}
                className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl font-medium text-sm transition-all ${currentTab === 'reports' ? 'bg-blue-600 text-white shadow-lg shadow-blue-500/15' : 'hover:bg-slate-150 dark:hover:bg-slate-800'}`}
              >
                <Icons.FileText />
                <span>Laporan & Analitis</span>
              </button>

              {userRole === 'Admin' && (
                <>
                  <div className="pt-4 pb-1 px-4 text-[10px] font-bold uppercase tracking-wider text-slate-400">
                    Menu Urus Tadbir
                  </div>
                  <button 
                    onClick={() => { setCurrentTab('manage-rooms'); setSidebarOpen(false); }}
                    className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl font-medium text-sm transition-all ${currentTab === 'manage-rooms' ? 'bg-emerald-600 text-white shadow-lg shadow-emerald-500/15' : 'hover:bg-slate-150 dark:hover:bg-slate-800'}`}
                  >
                    <Icons.Shield />
                    <span>Urus Bilik Khas</span>
                  </button>
                  <button 
                    onClick={() => { setCurrentTab('audit'); setSidebarOpen(false); }}
                    className={`w-full flex items-center space-x-3 px-4 py-3 rounded-xl font-medium text-sm transition-all ${currentTab === 'audit' ? 'bg-emerald-600 text-white shadow-lg shadow-emerald-500/15' : 'hover:bg-slate-150 dark:hover:bg-slate-800'}`}
                  >
                    <Icons.Info />
                    <span>Log Audit Sistem</span>
                  </button>
                </>
              )}

              {/* Info & Penafian Aplikasi */}
              <div className={`mt-8 p-4 rounded-2xl border text-xs leading-relaxed transition ${isDarkMode ? 'bg-[#1E293B]/40 border-slate-800 text-slate-400' : 'bg-slate-50 border-slate-200 text-slate-500'}`}>
                <p className="font-bold mb-1 flex items-center gap-1 text-slate-700 dark:text-slate-300">
                  <Icons.Info /> Petunjuk Akses
                </p>
                <p>Status peranan semasa anda adalah <strong>{userRole}</strong>. Jadual operasi adalah hari bekerja sekolah iaitu <strong>Ahad hingga Khamis</strong>.</p>
              </div>
            </nav>
          </aside>

          {/* MAIN CONTENT AREA */}
          <main className="lg:col-span-9">

            {/* TAB 1: DASHBOARD UTAMA */}
            {currentTab === 'dashboard' && (
              <div className="space-y-6 animate-fade-in">
                
                {/* Banner Aluan */}
                <div className="relative rounded-3xl overflow-hidden bg-gradient-to-r from-blue-600 to-indigo-700 text-white p-6 sm:p-8 shadow-xl">
                  <div className="relative z-10 max-w-lg">
                    <span className="bg-white/20 text-white text-xs font-bold px-3 py-1 rounded-full uppercase tracking-wider">
                      Sesi Persekolahan 2026
                    </span>
                    <h2 className="text-2xl sm:text-3xl font-extrabold mt-3">Sistem Pengurusan & Tempahan Bilik Khas</h2>
                    <p className="text-blue-100 mt-2 text-sm sm:text-base">
                      Semak kekosongan secara real-time, buat tempahan hari bekerja (Ahad - Khamis), dan elakkan pertembungan masa.
                    </p>
                    <div className="mt-5 flex gap-3">
                      <button 
                        onClick={() => setCurrentTab('booking-form')}
                        className="bg-white text-blue-600 font-bold px-5 py-2.5 rounded-xl hover:bg-slate-100 transition shadow-md"
                      >
                        Mula Tempah Sekarang
                      </button>
                      <button 
                        onClick={() => setCurrentTab('calendar')}
                        className="bg-blue-700/50 hover:bg-blue-700/70 text-white font-semibold px-4 py-2.5 rounded-xl transition"
                      >
                        Lihat Kalendar
                      </button>
                    </div>
                  </div>
                  {/* Decorative Abstract Background Shape */}
                  <div className="absolute right-0 bottom-0 top-0 w-1/3 bg-gradient-to-l from-white/10 to-transparent pointer-events-none hidden md:block"></div>
                </div>

                {/* Grid Statistik Dashboard */}
                <div className="grid grid-cols-2 lg:grid-cols-4 gap-4">
                  <div className={`p-5 rounded-2xl border transition shadow-sm ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-100'}`}>
                    <span className="text-xs text-slate-400 font-semibold uppercase tracking-wider block">Total Tempahan</span>
                    <span className="text-3xl font-bold mt-1 block">{stats.total}</span>
                    <span className="text-xs text-emerald-500 mt-2 block font-medium">Dalam Pangkalan Data</span>
                  </div>
                  <div className={`p-5 rounded-2xl border transition shadow-sm ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-100'}`}>
                    <span className="text-xs text-slate-400 font-semibold uppercase tracking-wider block">Menunggu Kelulusan</span>
                    <span className="text-3xl font-bold mt-1 block text-yellow-500">{stats.menunggu}</span>
                    <span className="text-xs text-slate-400 mt-2 block font-medium">Menunggu Tindakan Admin</span>
                  </div>
                  <div className={`p-5 rounded-2xl border transition shadow-sm ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-100'}`}>
                    <span className="text-xs text-slate-400 font-semibold uppercase tracking-wider block">Diluluskan</span>
                    <span className="text-3xl font-bold mt-1 block text-emerald-500">{stats.diluluskan}</span>
                    <span className="text-xs text-slate-400 mt-2 block font-medium">Sedia Untuk Digunakan</span>
                  </div>
                  <div className={`p-5 rounded-2xl border transition shadow-sm ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-100'}`}>
                    <span className="text-xs text-slate-400 font-semibold uppercase tracking-wider block">Jumlah Bilik Aktif</span>
                    <span className="text-3xl font-bold mt-1 block text-blue-500">{rooms.length}</span>
                    <span className="text-xs text-slate-400 mt-2 block font-medium">Bilik Khas Sekolah</span>
                  </div>
                </div>

                {/* Bahagian Utama Carian & Penapis */}
                <div className={`p-6 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                  <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-6">
                    <div>
                      <h3 className="text-lg font-bold">Semua Rekod Tempahan</h3>
                      <p className="text-xs text-slate-400">Gunakan carian pantas atau penapis untuk melihat status tertentu</p>
                    </div>
                    
                    <div className="flex flex-wrap gap-2 w-full md:w-auto">
                      <div className="relative flex-1 md:w-64">
                        <span className="absolute inset-y-0 left-0 flex items-center pl-3 text-slate-400 pointer-events-none">
                          <Icons.Search />
                        </span>
                        <input 
                          type="text" 
                          placeholder="Cari guru, bilik, tujuan..." 
                          value={searchQuery}
                          onChange={(e) => setSearchQuery(e.target.value)}
                          className={`w-full pl-10 pr-4 py-2 text-xs rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>

                      <select 
                        value={filterStatus} 
                        onChange={(e) => setFilterStatus(e.target.value)}
                        className={`text-xs px-3 py-2 rounded-xl border focus:outline-none ${isDarkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-50 border-slate-200'}`}
                      >
                        <option value="Semua">Semua Status</option>
                        <option value="Menunggu">Menunggu</option>
                        <option value="Diluluskan">Diluluskan</option>
                        <option value="Ditolak">Ditolak</option>
                      </select>

                      <select 
                        value={filterRoom} 
                        onChange={(e) => setFilterRoom(e.target.value)}
                        className={`text-xs px-3 py-2 rounded-xl border focus:outline-none ${isDarkMode ? 'bg-slate-800 border-slate-700' : 'bg-slate-50 border-slate-200'}`}
                      >
                        <option value="Semua">Semua Bilik</option>
                        {rooms.map(r => (
                          <option key={r.id} value={r.id}>{r.nama}</option>
                        ))}
                      </select>
                    </div>
                  </div>

                  {/* Senarai Tempahan Table */}
                  <div className="overflow-x-auto rounded-xl border border-slate-100 dark:border-slate-800">
                    <table className="w-full text-left text-xs border-collapse">
                      <thead>
                        <tr className={`${isDarkMode ? 'bg-slate-800/50' : 'bg-slate-50'} border-b border-slate-100 dark:border-slate-800`}>
                          <th className="p-4 font-bold text-slate-500 uppercase tracking-wider">Guru Pemohon</th>
                          <th className="p-4 font-bold text-slate-500 uppercase tracking-wider">Bilik Khas</th>
                          <th className="p-4 font-bold text-slate-500 uppercase tracking-wider">Tarikh / Masa</th>
                          <th className="p-4 font-bold text-slate-500 uppercase tracking-wider text-center">Status</th>
                          <th className="p-4 font-bold text-slate-500 uppercase tracking-wider text-right">Tindakan</th>
                        </tr>
                      </thead>
                      <tbody className="divide-y divide-slate-100 dark:divide-slate-800">
                        {filteredBookings.length === 0 ? (
                          <tr>
                            <td colSpan="5" className="p-8 text-center text-slate-400">
                              Tiada tempahan ditemui padanan penapis anda.
                            </td>
                          </tr>
                        ) : (
                          filteredBookings.map((b) => {
                            const bRoom = rooms.find(r => r.id === b.roomId);
                            return (
                              <tr key={b.id} className={`hover:bg-slate-50/50 dark:hover:bg-slate-800/20 transition`}>
                                <td className="p-4">
                                  <div>
                                    <p className="font-bold text-sm">{b.teacherName}</p>
                                    <p className="text-slate-400 text-[10px]">{b.jawatan}</p>
                                  </div>
                                </td>
                                <td className="p-4">
                                  <div className="font-semibold text-slate-700 dark:text-slate-200">{bRoom ? bRoom.nama : 'Bilik Dipadam'}</div>
                                  <p className="text-slate-400 text-[10px]">{bRoom?.lokasi}</p>
                                </td>
                                <td className="p-4">
                                  <div className="font-medium">{b.tarikh} ({b.hari})</div>
                                  <div className="text-blue-500 font-semibold">{b.masaMula} - {b.masaTamat}</div>
                                </td>
                                <td className="p-4 text-center">
                                  <span className={`inline-flex items-center px-2.5 py-1 rounded-full text-[10px] font-bold uppercase tracking-wider ${
                                    b.status === 'Diluluskan' ? 'bg-emerald-500/10 text-emerald-500' :
                                    b.status === 'Menunggu' ? 'bg-amber-500/10 text-amber-500' :
                                    b.status === 'Ditolak' ? 'bg-rose-500/10 text-rose-500' : 'bg-blue-500/10 text-blue-500'
                                  }`}>
                                    {b.status}
                                  </span>
                                </td>
                                <td className="p-4 text-right space-x-1">
                                  {/* Tindakan Admin */}
                                  {userRole === 'Admin' && b.status === 'Menunggu' && (
                                    <>
                                      <button 
                                        onClick={() => handleTukarStatus(b.id, 'Diluluskan')}
                                        className="p-1.5 rounded-lg bg-emerald-50 text-emerald-600 hover:bg-emerald-100 transition"
                                        title="Luluskan Tempahan"
                                      >
                                        <Icons.Check />
                                      </button>
                                      <button 
                                        onClick={() => handleTukarStatus(b.id, 'Ditolak')}
                                        className="p-1.5 rounded-lg bg-rose-50 text-rose-600 hover:bg-rose-100 transition"
                                        title="Tolak Tempahan"
                                      >
                                        <Icons.X />
                                      </button>
                                    </>
                                  )}
                                  
                                  {/* Print / View Slip */}
                                  <button 
                                    onClick={() => { setSelectedBookingDetails(b); setShowSlipModal(true); }}
                                    className="p-1.5 rounded-lg bg-slate-100 text-slate-600 hover:bg-slate-200 dark:bg-slate-800 dark:text-slate-300 transition"
                                    title="Cetak Slip Tempahan"
                                  >
                                    <Icons.Printer />
                                  </button>

                                  {/* Padam Tempahan */}
                                  {(userRole === 'Admin' || (b.teacherId === user?.uid && b.status === 'Menunggu')) && (
                                    <button 
                                      onClick={() => handleBatalTempahan(b.id)}
                                      className="p-1.5 rounded-lg bg-red-50 text-red-600 hover:bg-red-100 transition"
                                      title="Padam Tempahan"
                                    >
                                      <Icons.Trash />
                                    </button>
                                  )}
                                </td>
                              </tr>
                            );
                          })
                        )}
                      </tbody>
                    </table>
                  </div>
                </div>

              </div>
            )}

            {/* TAB 2: BORANG TEMPAHAN BARU */}
            {currentTab === 'booking-form' && (
              <div className="space-y-6 animate-fade-in">
                <div className={`p-6 sm:p-8 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                  
                  <div className="border-b pb-4 mb-6 border-slate-100 dark:border-slate-800">
                    <h2 className="text-xl font-bold bg-gradient-to-r from-blue-600 to-indigo-500 bg-clip-text text-transparent">Borang Tempahan Bilik Khas</h2>
                    <p className="text-xs text-slate-400 mt-1">Sila pilih tarikh bekerja (**Ahad hingga Khamis**) dan masa antara **7:30 Pagi hingga 4:30 Petang**.</p>
                  </div>

                  {/* Amaran Konflik Secara Live */}
                  {liveConflict && (
                    <div className="mb-6 p-4 rounded-2xl bg-rose-500/10 border border-rose-500/30 text-rose-500 flex items-start gap-3">
                      <div className="mt-0.5"><Icons.X /></div>
                      <div className="text-xs">
                        <p className="font-bold">Pertembungan Waktu Dikesan!</p>
                        <p className="mt-1">
                          Bilik ini telah ditempah oleh <strong>{liveConflict.teacherName}</strong> pada <strong>{formData.tarikh}</strong> pada jam <strong>{liveConflict.masaMula} hingga {liveConflict.masaTamat}</strong>.
                        </p>
                        <p className="font-semibold mt-1">Sila tukar tarikh atau slot masa sebelum meneruskan.</p>
                      </div>
                    </div>
                  )}

                  <form onSubmit={handleHantarTempahan} className="space-y-5 text-xs">
                    
                    {/* Seksyen 1: Maklumat Guru */}
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Nama Guru Pemohon</label>
                        <input 
                          type="text" 
                          required
                          value={formData.teacherName}
                          onChange={(e) => setFormData({...formData, teacherName: e.target.value})}
                          placeholder="Masukkan nama penuh guru"
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">No. Telefon Bimbit</label>
                        <input 
                          type="text" 
                          required
                          value={formData.teacherPhone}
                          onChange={(e) => setFormData({...formData, teacherPhone: e.target.value})}
                          placeholder="Contoh: 0123456789"
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>
                    </div>

                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">E-mel DELIMa</label>
                        <input 
                          type="email" 
                          required
                          value={formData.teacherEmail}
                          onChange={(e) => setFormData({...formData, teacherEmail: e.target.value})}
                          placeholder="alamat@moe-dl.edu.my"
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Jawatan</label>
                        <select 
                          value={formData.jawatan}
                          onChange={(e) => setFormData({...formData, jawatan: e.target.value})}
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        >
                          <option>Guru Akademik Biasa</option>
                          <option>Ketua Panitia</option>
                          <option>Guru Kanan Mata Pelajaran</option>
                          <option>Penyelaras Bilik Khas</option>
                        </select>
                      </div>
                    </div>

                    {/* Seksyen 2: Maklumat Slot Tempahan */}
                    <div className="grid grid-cols-1 md:grid-cols-4 gap-4 pt-4 border-t border-slate-100 dark:border-slate-800">
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Pilih Bilik Khas</label>
                        <select 
                          value={formData.roomId}
                          onChange={(e) => setFormData({...formData, roomId: e.target.value})}
                          className={`w-full p-3 rounded-xl border font-bold focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        >
                          {rooms.map(r => (
                            <option key={r.id} value={r.id}>{r.nama}</option>
                          ))}
                        </select>
                      </div>
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Tarikh Penggunaan</label>
                        <input 
                          type="date" 
                          required
                          value={formData.tarikh}
                          onChange={(e) => setFormData({...formData, tarikh: e.target.value})}
                          className={`w-full p-3 rounded-xl border font-bold focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Hari (Automatik)</label>
                        <input 
                          type="text" 
                          readOnly
                          value={formData.hari}
                          className="w-full p-3 rounded-xl border bg-slate-100 dark:bg-slate-800 font-bold outline-none cursor-not-allowed text-slate-500"
                        />
                      </div>
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Jumlah Murid Terlibat</label>
                        <input 
                          type="number" 
                          required
                          min="1"
                          value={formData.bilMurid}
                          onChange={(e) => setFormData({...formData, bilMurid: e.target.value})}
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>
                    </div>

                    <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Masa Mula</label>
                        <select 
                          value={formData.masaMula}
                          onChange={(e) => setFormData({...formData, masaMula: e.target.value})}
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        >
                          {TIME_SLOTS.map(t => (
                            <option key={t} value={t}>{t}</option>
                          ))}
                        </select>
                      </div>
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Masa Tamat</label>
                        <select 
                          value={formData.masaTamat}
                          onChange={(e) => setFormData({...formData, masaTamat: e.target.value})}
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        >
                          {TIME_SLOTS.map(t => (
                            <option key={t} value={t}>{t}</option>
                          ))}
                        </select>
                      </div>
                    </div>

                    <div className="pt-4 border-t border-slate-100 dark:border-slate-800">
                      <div>
                        <label className="block text-slate-400 font-semibold mb-1.5">Tujuan / Aktiviti Pembelajaran</label>
                        <input 
                          type="text" 
                          required
                          value={formData.tujuan}
                          onChange={(e) => setFormData({...formData, tujuan: e.target.value})}
                          placeholder="Contoh: Amali Sains Komputer Bab 3"
                          className={`w-full p-3 rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                        />
                      </div>
                    </div>

                    {/* Actions Panel */}
                    <div className="flex justify-end space-x-3 pt-4">
                      <button 
                        type="button"
                        onClick={() => setFormData(prev => ({
                          ...prev,
                          tujuan: '',
                          bilMurid: 30
                        }))}
                        className={`px-5 py-3 rounded-xl font-bold transition ${isDarkMode ? 'bg-slate-800 hover:bg-slate-700' : 'bg-slate-100 hover:bg-slate-200'}`}
                      >
                        Set Semula
                      </button>
                      <button 
                        type="submit"
                        disabled={!!liveConflict}
                        className={`px-6 py-3 rounded-xl font-bold text-white shadow-md transition ${liveConflict ? 'bg-slate-400 cursor-not-allowed shadow-none' : 'bg-blue-600 hover:bg-blue-700'}`}
                      >
                        Hantar Tempahan Bilik
                      </button>
                    </div>

                  </form>
                </div>
              </div>
            )}

            {/* TAB 3: KALENDAR SLOT MASA */}
            {currentTab === 'calendar' && (
              <div className="space-y-6 animate-fade-in">
                <div className={`p-6 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                  
                  <div className="flex flex-col md:flex-row justify-between items-start md:items-center gap-4 mb-6">
                    <div>
                      <h2 className="text-xl font-bold">Kalendar & Waktu Slot Operasi</h2>
                      <p className="text-xs text-slate-400">Jadual visual penggunaan bilik khas di sekolah (Ahad - Khamis, 7:30 Pagi - 4:30 Petang)</p>
                    </div>
                    
                    {/* Petunjuk Warna Kalendar */}
                    <div className="flex flex-wrap gap-3 text-xs font-semibold">
                      <div className="flex items-center space-x-1.5">
                        <span className="w-3.5 h-3.5 rounded bg-emerald-500"></span>
                        <span>Slot Kosong</span>
                      </div>
                      <div className="flex items-center space-x-1.5">
                        <span className="w-3.5 h-3.5 rounded bg-rose-500"></span>
                        <span>Slot Ditempah</span>
                      </div>
                      <div className="flex items-center space-x-1.5">
                        <span className="w-3.5 h-3.5 rounded bg-slate-400"></span>
                        <span>Cuti Hujung Minggu</span>
                      </div>
                    </div>
                  </div>

                  {/* Kalendar Grid */}
                  <div className="grid grid-cols-7 gap-1.5 mb-6 text-center text-xs">
                    {['Ahad', 'Isnin', 'Selasa', 'Rabu', 'Khamis', 'Jumaat', 'Sabtu'].map(d => (
                      <div key={d} className="font-bold text-slate-400 py-2">{d}</div>
                    ))}
                    
                    {Array.from({ length: 28 }).map((_, i) => {
                      const dayNumber = i + 1;
                      const dayString = `2026-08-${dayNumber.toString().padStart(2, '0')}`;
                      const dayName = getMalayDayName(dayString);
                      // Cuti hujung minggu diubah kepada Jumaat & Sabtu
                      const isWeekend = dayName === 'Jumaat' || dayName === 'Sabtu';
                      
                      const daysBookings = bookings.filter(b => b.tarikh === dayString && b.status !== 'Ditolak');
                      const hasBookings = daysBookings.length > 0;

                      return (
                        <div 
                          key={i} 
                          onClick={() => {
                            if (!isWeekend) {
                              setFormData(prev => ({ ...prev, tarikh: dayString }));
                              setCurrentTab('booking-form');
                            }
                          }}
                          className={`p-3 rounded-xl border flex flex-col items-center justify-between min-h-[70px] cursor-pointer transition ${
                            isWeekend ? 'bg-slate-150/40 dark:bg-slate-800/40 border-dashed text-slate-400' :
                            hasBookings ? 'bg-rose-500/10 border-rose-500/30 text-rose-500 hover:bg-rose-500/20' :
                            'bg-emerald-500/5 border-emerald-500/20 text-emerald-600 hover:bg-emerald-500/15'
                          }`}
                        >
                          <span className="font-bold text-xs">{dayNumber}</span>
                          <span className="text-[9px] mt-2 font-semibold">
                            {isWeekend ? 'Cuti' : hasBookings ? `${daysBookings.length} Tempah` : 'Kosong'}
                          </span>
                        </div>
                      );
                    })}
                  </div>

                  {/* Ringkasan Maklumat Slot Masa Hari Ini */}
                  <div className="border-t pt-5 border-slate-150 dark:border-slate-800">
                    <h3 className="font-bold text-sm mb-3">Senarai Slot Terkunci Untuk Tarikh: <span className="text-blue-500">{formData.tarikh}</span></h3>
                    
                    <div className="grid grid-cols-2 sm:grid-cols-4 md:grid-cols-6 gap-2">
                      {TIME_SLOTS.map((t, idx) => {
                        const tempahanDislot = bookings.find(b => 
                          b.tarikh === formData.tarikh && 
                          b.masaMula <= t && b.masaTamat > t &&
                          b.status !== 'Ditolak'
                        );

                        return (
                          <div 
                            key={idx}
                            className={`p-2.5 rounded-xl text-center border text-xs font-semibold ${
                              tempahanDislot 
                                ? 'bg-rose-100 border-rose-200 text-rose-700 dark:bg-rose-900/30 dark:border-rose-900/50 dark:text-rose-400' 
                                : 'bg-emerald-50 border-emerald-100 text-emerald-700 dark:bg-emerald-900/10 dark:border-emerald-900/30 dark:text-emerald-400'
                            }`}
                          >
                            <div>{t}</div>
                            <div className="text-[9px] mt-1 font-medium truncate">
                              {tempahanDislot ? tempahanDislot.teacherName.split(' ')[1] || 'Ditempah' : 'Sedia Diambil'}
                            </div>
                          </div>
                        );
                      })}
                    </div>
                  </div>

                </div>
              </div>
            )}

            {/* TAB 4: LAPORAN & ANALITIS */}
            {currentTab === 'reports' && (
              <div className="space-y-6 animate-fade-in printable">
                
                {/* Kawalan Eksport Laporan */}
                <div className="flex flex-wrap justify-between items-center gap-3 bg-gradient-to-tr from-slate-850 to-slate-900 p-4 rounded-2xl border dark:border-slate-800">
                  <div>
                    <h3 className="font-bold text-sm">Eksport Dokumen Laporan</h3>
                    <p className="text-[11px] text-slate-400">Muat turun data pangkalan data untuk simpanan arkib sekolah.</p>
                  </div>
                  <div className="flex gap-2">
                    <button 
                      onClick={handleExportExcel}
                      className="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl shadow-md flex items-center gap-1.5 transition"
                    >
                      <Icons.FileText /> Eksport Excel/CSV
                    </button>
                    <button 
                      onClick={handleCetakLaporan}
                      className="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white font-bold text-xs rounded-xl shadow-md flex items-center gap-1.5 transition"
                    >
                      <Icons.Printer /> Cetak Laporan PDF
                    </button>
                  </div>
                </div>

                {/* Ringkasan Statistik Laporan */}
                <div className="grid grid-cols-1 gap-6">
                  
                  {/* Statistik Mengikut Bilik */}
                  <div className={`p-6 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                    <h3 className="font-bold text-sm mb-4">Grafik Penggunaan Bilik Khas</h3>
                    <div className="space-y-3">
                      {rooms.map(r => {
                        const count = stats.bilikCounts[r.nama] || 0;
                        const peratus = stats.total > 0 ? (count / stats.total) * 100 : 0;
                        return (
                          <div key={r.id} className="text-xs">
                            <div className="flex justify-between items-center mb-1 font-medium">
                              <span>{r.nama}</span>
                              <span className="font-bold">{count} Kali ({peratus.toFixed(0)}%)</span>
                            </div>
                            <div className="w-full bg-slate-100 dark:bg-slate-800 h-2 rounded-full overflow-hidden">
                              <div className="bg-blue-600 h-2 rounded-full" style={{ width: `${peratus}%` }}></div>
                            </div>
                          </div>
                        );
                      })}
                    </div>
                  </div>

                </div>

              </div>
            )}

            {/* TAB 5: URUS BILIK (Admin sahaja) */}
            {currentTab === 'manage-rooms' && userRole === 'Admin' && (
              <div className="space-y-6 animate-fade-in">
                
                {/* Tambah Bilik Baru Form */}
                <div className={`p-6 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                  <h3 className="font-bold text-sm mb-3">Pendaftaran Bilik Khas Baharu</h3>
                  <form onSubmit={handleTambahBilik} className="grid grid-cols-1 sm:grid-cols-3 gap-3 items-end">
                    <div>
                      <label className="block text-slate-400 text-xs font-semibold mb-1">Nama Bilik Khas</label>
                      <input 
                        type="text" 
                        required
                        value={newRoomName}
                        onChange={(e) => setNewRoomName(e.target.value)}
                        placeholder="Contoh: Makmal Komputer 3"
                        className={`w-full p-2.5 text-xs rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                      />
                    </div>
                    <div>
                      <label className="block text-slate-400 text-xs font-semibold mb-1">Lokasi Blok & Aras</label>
                      <input 
                        type="text" 
                        required
                        value={newRoomLoc}
                        onChange={(e) => setNewRoomLoc(e.target.value)}
                        placeholder="Contoh: Blok B, Aras G"
                        className={`w-full p-2.5 text-xs rounded-xl border focus:outline-none focus:ring-2 focus:ring-blue-500 ${isDarkMode ? 'bg-slate-800 border-slate-700 text-white' : 'bg-slate-50 border-slate-200'}`}
                      />
                    </div>
                    <button 
                      type="submit"
                      className="p-2.5 bg-blue-600 hover:bg-blue-700 text-white font-bold text-xs rounded-xl shadow transition flex justify-center items-center gap-1"
                    >
                      <Icons.Plus /> Daftarkan Bilik
                    </button>
                  </form>
                </div>

                {/* Senarai Urus Bilik */}
                <div className={`p-6 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                  <h3 className="font-bold text-sm mb-4">Senarai Bilik Berdaftar</h3>
                  <div className="grid grid-cols-1 sm:grid-cols-2 gap-4">
                    {rooms.map(r => (
                      <div key={r.id} className="p-4 rounded-2xl border border-slate-100 dark:border-slate-800 flex justify-between items-center bg-slate-50/50 dark:bg-slate-800/10">
                        <div>
                          <p className="font-bold text-sm">{r.nama}</p>
                          <p className="text-slate-400 text-xs">{r.lokasi}</p>
                        </div>
                        <button 
                          onClick={() => handlePadamBilik(r.id)}
                          className="p-2 bg-red-50 text-red-500 rounded-xl hover:bg-red-100 transition"
                          title="Padam Bilik"
                        >
                          <Icons.Trash />
                        </button>
                      </div>
                    ))}
                  </div>
                </div>

              </div>
            )}

            {/* TAB 6: LOG AUDIT SISTEM */}
            {currentTab === 'audit' && userRole === 'Admin' && (
              <div className="space-y-6 animate-fade-in">
                <div className={`p-6 rounded-3xl border transition ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-150'}`}>
                  <div className="flex justify-between items-center mb-4">
                    <div>
                      <h3 className="font-bold text-sm">Audit Log Jejak Pengguna</h3>
                      <p className="text-xs text-slate-400">Jejak sejarah aktiviti tempahan dan tindakan konfigurasi pangkalan data.</p>
                    </div>
                    <button 
                      onClick={() => setAuditLogs([])} 
                      className="text-xs text-red-500 hover:underline"
                    >
                      Kosongkan Sejarah Log
                    </button>
                  </div>

                  <div className="space-y-2.5 max-h-[500px] overflow-y-auto pr-2">
                    {auditLogs.map(log => (
                      <div key={log.id} className="p-3 text-xs rounded-xl bg-slate-50 dark:bg-slate-800/30 flex justify-between items-center border dark:border-slate-850">
                        <div>
                          <span className="font-bold text-blue-500">{log.pengguna}</span>
                          <span className="mx-2 text-slate-300">|</span>
                          <span className="text-slate-600 dark:text-slate-300">{log.tindakan}</span>
                        </div>
                        <span className="text-[10px] text-slate-400 font-semibold">{log.timestamp}</span>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            )}

          </main>
        </div>
      </div>

      {/* FOOTER */}
      <footer className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-10 mt-12 border-t border-slate-200 dark:border-slate-800 text-center text-xs text-slate-400">
        <p>© 2026 Sistem Tempahan Bilik Khas Sekolah (STBKS). Hak Cipta Terpelihara.</p>
        <p className="mt-1">Dibangunkan dengan ciri kawalan konflik automatik, Firestore, dan antara muka moden tanpa ruangan luaran tidak perlu.</p>
      </footer>

      {/* MODAL FEEDBACK TEMPAHAN */}
      {modalFeedback && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm">
          <div className={`max-w-md w-full p-6 rounded-3xl shadow-2xl border text-center ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-100'}`}>
            <div className={`mx-auto w-12 h-12 rounded-full flex items-center justify-center text-white mb-4 ${
              modalFeedback.type === 'success' ? 'bg-emerald-500' : 'bg-rose-500'
            }`}>
              {modalFeedback.type === 'success' ? <Icons.Check /> : <Icons.X />}
            </div>
            
            <h3 className="font-bold text-lg mb-2">
              {modalFeedback.type === 'success' ? 'Tempahan Selesai!' : 'Ralat Tempahan'}
            </h3>
            <p className="text-xs text-slate-400 leading-relaxed mb-6">{modalFeedback.message}</p>
            
            <button 
              onClick={() => setModalFeedback(null)}
              className="w-full py-3 bg-blue-600 text-white text-xs font-bold rounded-xl hover:bg-blue-700 transition"
            >
              Faham, Tutup Semakan
            </button>
          </div>
        </div>
      )}

      {/* MODAL SLIP & CETAKAN TEMPAHAN */}
      {showSlipModal && selectedBookingDetails && (
        <div className="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/60 backdrop-blur-sm">
          <div className={`max-w-lg w-full p-6 rounded-3xl shadow-2xl border ${isDarkMode ? 'bg-[#1E293B] border-slate-800' : 'bg-white border-slate-100'}`}>
            
            <div className="flex justify-between items-center border-b pb-3 mb-4 dark:border-slate-800">
              <h3 className="font-bold text-sm">Slip Cetakan Tempahan Rasmi</h3>
              <button onClick={() => setShowSlipModal(false)} className="p-1 hover:bg-slate-100 dark:hover:bg-slate-800 rounded-lg">
                <Icons.X />
              </button>
            </div>

            <div className="space-y-4 text-xs">
              <div className="flex justify-between">
                <span className="text-slate-400">Nama Guru:</span>
                <span className="font-bold">{selectedBookingDetails.teacherName}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">E-mel DELIMa:</span>
                <span className="font-semibold">{selectedBookingDetails.teacherEmail}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">Bilik Khas:</span>
                <span className="font-bold text-blue-500">{rooms.find(r => r.id === selectedBookingDetails.roomId)?.nama || 'Bilik Khas'}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">Tarikh Penggunaan:</span>
                <span className="font-bold">{selectedBookingDetails.tarikh} ({selectedBookingDetails.hari})</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">Slot Masa:</span>
                <span className="font-bold text-emerald-500">{selectedBookingDetails.masaMula} - {selectedBookingDetails.masaTamat}</span>
              </div>
              <div className="flex justify-between">
                <span className="text-slate-400">Bilangan Murid:</span>
                <span className="font-semibold">{selectedBookingDetails.bilMurid} Murid</span>
              </div>
              <div className="border-t pt-3 dark:border-slate-800">
                <span className="text-slate-400 block mb-1">Tujuan Tempahan:</span>
                <p className="p-2.5 rounded-lg bg-slate-50 dark:bg-slate-800 font-medium">{selectedBookingDetails.tujuan}</p>
              </div>

              {/* QR Code Pembantu Simulasi */}
              <div className="flex items-center space-x-3 p-3 rounded-2xl bg-blue-50/50 dark:bg-blue-900/20 border border-blue-500/10">
                <div className="w-16 h-16 bg-white border flex items-center justify-center p-1 rounded-lg">
                  <svg viewBox="0 0 100 100" className="w-full h-full text-slate-800">
                    <rect x="0" y="0" width="30" height="30" fill="currentColor" />
                    <rect x="70" y="0" width="30" height="30" fill="currentColor" />
                    <rect x="0" y="70" width="30" height="30" fill="currentColor" />
                    <rect x="10" y="10" width="10" height="10" fill="white" />
                    <rect x="80" y="10" width="10" height="10" fill="white" />
                    <rect x="10" y="80" width="10" height="10" fill="white" />
                    <rect x="40" y="40" width="20" height="20" fill="currentColor" />
                    <rect x="50" y="10" width="10" height="20" fill="currentColor" />
                    <rect x="10" y="50" width="20" height="10" fill="currentColor" />
                  </svg>
                </div>
                <div>
                  <p className="font-bold text-[11px]">QR Pengesahan Slip</p>
                  <p className="text-[10px] text-slate-400 leading-normal">Imbas menggunakan telefon pintar untuk menyemak ketulenan tempahan.</p>
                </div>
              </div>
            </div>

            <div className="mt-6 flex gap-2">
              <button 
                onClick={() => { window.print(); writeLog("Mencetak slip tempahan"); }}
                className="flex-1 py-3 bg-blue-600 hover:bg-blue-700 text-white text-xs font-bold rounded-xl transition flex justify-center items-center gap-1.5"
              >
                <Icons.Printer /> Cetak Slip
              </button>
              <button 
                onClick={() => setShowSlipModal(false)}
                className="px-4 py-3 bg-slate-100 hover:bg-slate-200 dark:bg-slate-800 dark:hover:bg-slate-700 text-xs font-bold rounded-xl transition"
              >
                Tutup
              </button>
            </div>

          </div>
        </div>
      )}

    </div>
  );
}
