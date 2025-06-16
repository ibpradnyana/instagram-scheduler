# instagram-scheduler
For Instagram Content Schedule
// src/App.jsx
import { WeeklyContentScheduler } from './components/WeeklyContentScheduler';

function App() {
  return (
    <div className="min-h-screen bg-gray-50 p-4 md:p-8">
      <WeeklyContentScheduler />
    </div>
  );
}

export default App;

// src/components/WeeklyContentScheduler.jsx
import { useState, useEffect } from 'react';

const scheduleTemplate = {
  Senin: 'Produk',
  Selasa: 'Klaim',
  Rabu: 'Bisnis',
  Kamis: 'Bisnis',
  Jumat: 'Mitos/Fakta/Quote',
  Sabtu: 'Kuis',
  Minggu: 'Tips/Informasi',
};

function getWeekRange(startDate) {
  const day = startDate.getDay();
  const diffToMonday = (day + 6) % 7;
  const monday = new Date(startDate);
  monday.setDate(startDate.getDate() - diffToMonday);
  const sunday = new Date(monday);
  sunday.setDate(monday.getDate() + 6);

  const options = { weekday: 'long', day: 'numeric', month: 'long', year: 'numeric' };
  const start = monday.toLocaleDateString('id-ID', options);
  const end = sunday.toLocaleDateString('id-ID', options);
  return {
    range: `${start} - ${end}`,
    key: `${monday.getFullYear()}-${monday.getMonth()}-${monday.getDate()}`,
    monday,
  };
}

export function WeeklyContentScheduler() {
  const [currentMonday, setCurrentMonday] = useState(() => {
    const now = new Date();
    const day = now.getDay();
    const diffToMonday = (day + 6) % 7;
    const monday = new Date(now);
    monday.setDate(now.getDate() - diffToMonday);
    monday.setHours(0, 0, 0, 0);
    return monday;
  });

  const { range, key } = getWeekRange(currentMonday);

  const [content, setContent] = useState(() => {
    const saved = localStorage.getItem(`weeklyContent-${key}`);
    return saved ? JSON.parse(saved) : {};
  });

  useEffect(() => {
    const saved = localStorage.getItem(`weeklyContent-${key}`);
    setContent(saved ? JSON.parse(saved) : {});
  }, [key]);

  useEffect(() => {
    localStorage.setItem(`weeklyContent-${key}`, JSON.stringify(content));
  }, [content, key]);

  const handleChange = (day, value) => {
    const updatedContent = { ...content, [day]: value };
    setContent(updatedContent);
  };

  const clearAll = () => {
    setContent({});
    localStorage.removeItem(`weeklyContent-${key}`);
  };

  const changeWeek = (direction) => {
    const newMonday = new Date(currentMonday);
    newMonday.setDate(currentMonday.getDate() + direction * 7);
    setCurrentMonday(newMonday);
  };

  return (
    <div className="p-2 sm:p-4">
      <h1 className="text-2xl font-bold mb-2 text-center">Jadwal Konten Instagram Mingguan</h1>
      <div className="text-center mb-4">
        <button
          onClick={() => changeWeek(-1)}
          className="inline-block bg-gray-200 text-gray-700 px-4 py-1 rounded-xl mr-2 hover:bg-gray-300"
        >
          &laquo; Minggu Sebelumnya
        </button>
        <button
          onClick={() => changeWeek(1)}
          className="inline-block bg-gray-200 text-gray-700 px-4 py-1 rounded-xl hover:bg-gray-300"
        >
          Minggu Berikutnya &raquo;
        </button>
      </div>
      <p className="text-center text-gray-600 mb-6">{range}</p>
      <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
        {Object.entries(scheduleTemplate).map(([day, category]) => (
          <div key={day} className="border border-gray-300 p-4 rounded-2xl shadow-md bg-white">
            <h2 className="font-semibold text-lg mb-2">
              {day} - <span className="text-blue-600">{category}</span>
            </h2>
            <textarea
              className="w-full mt-1 p-3 border border-gray-200 rounded-xl text-sm sm:text-base focus:outline-none focus:ring-2 focus:ring-blue-500"
              rows="4"
              placeholder={`Isi konten untuk ${day}`}
              value={content[day] || ''}
              onChange={(e) => handleChange(day, e.target.value)}
            />
          </div>
        ))}
      </div>
      <div className="text-center mt-6">
        <button
          onClick={clearAll}
          className="bg-red-500 text-white px-6 py-2 rounded-xl text-sm sm:text-base hover:bg-red-600 transition"
        >
          Reset Mingguan
        </button>
      </div>
      <div className="mt-10">
        <h3 className="text-xl font-semibold mb-2">Preview Mingguan</h3>
        <div className="bg-gray-100 p-4 rounded-xl text-sm sm:text-base whitespace-pre-wrap">
          {Object.entries(scheduleTemplate).map(([day, _]) => (
            <div key={day} className="mb-2">
              <strong>{day}:</strong> {content[day] || '[Belum diisi]'}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
