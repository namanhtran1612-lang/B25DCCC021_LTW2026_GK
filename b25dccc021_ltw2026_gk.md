# TONG HOP CODE CHI TIET

### 1. `student.json` (Thu muc goc project)
```json
{
  "studentId": "B25DCCC010",
  "fullName": "Nguyen Van A",
  "className": "D25CQCC01-B",
  "examCode": "DE6"
}
```

---

### 2. `public/data/events.json`
```json
[
  { "id": 1, "name": "Hoi thao ReactJS co ban", "location": "Hoi truong A", "date": "2026-11-05", "capacity": 100, "registered": 85 },
  { "id": 2, "name": "Workshop Git va GitHub", "location": "Phong 302", "date": "2026-11-12", "capacity": 40, "registered": 40 },
  { "id": 3, "name": "Cuoc thi lap trinh PTIT Code", "location": "Phong may 1", "date": "2026-11-20", "capacity": 60, "registered": 12 },
  { "id": 4, "name": "Talkshow Dinh huong nghe nghiep", "location": "Hoi truong B", "date": "2026-12-01", "capacity": 200, "registered": 150 },
  { "id": 5, "name": "Ngay hoi viec lam IT", "location": "San truong", "date": "2026-12-10", "capacity": 500, "registered": 500 },
  { "id": 6, "name": "Seminar Bao mat Web", "location": "Phong 205", "date": "2027-01-08", "capacity": 50, "registered": 20 }
]
```

---

### 3. `src/index.js`
```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom';
import './App.css';
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

---

### 4. `src/App.css`
```css
.active {
  color: red;
  font-weight: bold;
}

nav a {
  margin-right: 15px;
  text-decoration: none;
  color: black;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

tr:hover {
  background-color: #f1f1f1;
}

.full-event {
  color: gray;
  font-style: italic;
}

.error-msg {
  color: red;
  margin-top: 10px;
}
```

---

### 5. `src/App.js`
```jsx
import React, { useState, useEffect } from 'react';
import { Routes, Route } from 'react-router-dom';
import axios from 'axios';
import Layout from './components/Layout';
import Home from './pages/Home';
import EventList from './pages/EventList';
import AddEvent from './pages/AddEvent';
import EventDetail from './pages/EventDetail';

const App = () => {
  const [events, setEvents] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(false);

  useEffect(() => {
    const controller = new AbortController();
    const savedEvents = localStorage.getItem('gk_events');

    if (savedEvents) {
      try {
        setEvents(JSON.parse(savedEvents));
        setLoading(false);
      } catch (err) {
        console.error(err);
      }
    } else {
      axios
        .get('/data/events.json', { signal: controller.signal })
        .then((res) => {
          setEvents(res.data);
          setLoading(false);
        })
        .catch((err) => {
          if (!axios.isCancel(err)) {
            setError(true);
            setLoading(false);
          }
        });
    }

    return () => {
      controller.abort();
    };
  }, []);

  useEffect(() => {
    if (!loading) {
      localStorage.setItem('gk_events', JSON.stringify(events));
    }
  }, [events, loading]);

  const handleDelete = (id) => {
    const updatedEvents = events.filter((ev) => ev.id !== id);
    setEvents(updatedEvents);
  };

  const handleRegister = (id) => {
    const updatedEvents = events.map((ev) => {
      if (ev.id === id && ev.registered < ev.capacity) {
        return { ...ev, registered: ev.registered + 1 };
      }
      return ev;
    });
    setEvents(updatedEvents);
  };

  const handleAdd = (newEvent) => {
    setEvents([...events, newEvent]);
  };

  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route
          index
          element={
            loading ? (
              <div>Dang tai...</div>
            ) : error ? (
              <div>Khong tai duoc du lieu</div>
            ) : (
              <Home events={events} />
            )
          }
        />
        <Route
          path="events"
          element={
            loading ? (
              <div>Dang tai...</div>
            ) : error ? (
              <div>Khong tai duoc du lieu</div>
            ) : (
              <EventList
                events={events}
                onDelete={handleDelete}
                onRegister={handleRegister}
              />
            )
          }
        />
        <Route
          path="events/new"
          element={<AddEvent events={events} onAdd={handleAdd} />}
        />
        <Route
          path="events/:id"
          element={
            loading ? (
              <div>Dang tai...</div>
            ) : error ? (
              <div>Khong tai duoc du lieu</div>
            ) : (
              <EventDetail events={events} />
            )
          }
        />
      </Route>
    </Routes>
  );
};

export default App;
```

---

### 6. `src/components/Layout.js`
```jsx
import React from 'react';
import { Outlet } from 'react-router-dom';
import NavBar from './NavBar';

const Layout = () => {
  return (
    <div style={{ padding: '20px', fontFamily: 'Arial, sans-serif' }}>
      <NavBar />
      <Outlet />
    </div>
  );
};

export default Layout;
```

---

### 7. `src/components/NavBar.js`
```jsx
import React from 'react';
import { NavLink, useLocation } from 'react-router-dom';

const NavBar = () => {
  const { pathname } = useLocation();
  const isDetail = pathname.startsWith('/events/') && pathname !== '/events' && pathname !== '/events/new';

  return (
    <nav style={{ marginBottom: '20px', borderBottom: '1px solid #ccc', paddingBottom: '10px' }}>
      <NavLink to="/" end className={({ isActive }) => (isDetail ? '' : isActive ? 'active' : '')}>
        Trang chu
      </NavLink>
      <NavLink to="/events" className={({ isActive }) => (isDetail ? '' : isActive ? 'active' : '')}>
        Danh sach
      </NavLink>
      <NavLink to="/events/new" className={({ isActive }) => (isDetail ? '' : isActive ? 'active' : '')}>
        Them moi
      </NavLink>
    </nav>
  );
};

export default NavBar;
```

---

### 8. `src/components/FilterBar.js`
```jsx
import React from 'react';

const FilterBar = ({ search, setSearch, filterStatus, setFilterStatus }) => {
  return (
    <div style={{ marginBottom: '15px' }}>
      <label htmlFor="searchInput">Tim kiem: </label>
      <input
        id="searchInput"
        type="text"
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Nhap ten su kien..."
        style={{ marginRight: '20px' }}
      />

      <label htmlFor="statusFilter">Tinh trang: </label>
      <select
        id="statusFilter"
        value={filterStatus}
        onChange={(e) => setFilterStatus(e.target.value)}
      >
        <option value="Tat ca">Tat ca</option>
        <option value="Con cho">Con cho</option>
        <option value="Da day">Da day</option>
      </select>
    </div>
  );
};

export default FilterBar;
```

---

### 9. `src/components/EventTable.js`
```jsx
import React from 'react';
import EventRow from './EventRow';

const EventTable = ({ events, onDelete, onRegister }) => {
  return (
    <table>
      <thead>
        <tr>
          <th>Su kien</th>
          <th>Dia diem</th>
          <th>Ngay</th>
          <th>So cho</th>
          <th>Thao tac</th>
        </tr>
      </thead>
      <tbody>
        {events.map((event) => (
          <EventRow key={event.id} event={event} onDelete={onDelete} onRegister={onRegister} />
        ))}
      </tbody>
    </table>
  );
};

export default EventTable;
```

---

### 10. `src/components/EventRow.js`
```jsx
import React from 'react';
import { Link } from 'react-router-dom';

const EventRow = ({ event, onDelete, onRegister }) => {
  const { id, name, location, date, capacity, registered } = event;
  const isFull = registered >= capacity;

  return (
    <tr>
      <td>
        <Link to={`/events/${id}`}>{name}</Link>
      </td>
      <td>{location}</td>
      <td>{date}</td>
      <td className={isFull ? 'full-event' : ''}>
        {registered}/{capacity}
      </td>
      <td>
        <button onClick={() => onRegister(id)} disabled={isFull}>
          Dang ky
        </button>{' '}
        <button onClick={() => onDelete(id)}>Xoa</button>
      </td>
    </tr>
  );
};

export default EventRow;
```

---

### 11. `src/components/Countdown.js`
```jsx
import React, { useState, useEffect } from 'react';

const Countdown = ({ date }) => {
  const [timeLeft, setTimeLeft] = useState('');

  useEffect(() => {
    const targetDate = new Date(`${date}T00:00:00`);

    const updateCountdown = () => {
      const now = new Date();
      const difference = targetDate - now;

      if (difference <= 0) {
        setTimeLeft('Su kien da dien ra');
        return;
      }

      const days = Math.floor(difference / (1000 * 60 * 60 * 24));
      const hours = Math.floor((difference / (1000 * 60 * 60)) % 24);
      const minutes = Math.floor((difference / 1000 / 60) % 60);
      const seconds = Math.floor((difference / 1000) % 60);

      setTimeLeft(`Con ${days} ngay ${hours} gio ${minutes} phut ${seconds} giay`);
    };

    updateCountdown();
    const interval = setInterval(updateCountdown, 1000);

    return () => clearInterval(interval);
  }, [date]);

  return <span>{timeLeft}</span>;
};

export default Countdown;
```

---

### 12. `src/pages/Home.js`
```jsx
import React from 'react';

const Home = ({ events }) => {
  const totalEvents = events.length;

  const totalRegistered = events.reduce((sum, ev) => sum + ev.registered, 0);

  const totalFullEvents = events.reduce((sum, ev) => {
    return ev.registered >= ev.capacity ? sum + 1 : sum;
  }, 0);

  return (
    <div>
      <h1>Su kien CLB</h1>
      <p>Tong so su kien: {totalEvents}</p>
      <p>Tong so luot dang ky: {totalRegistered}</p>
      <p>So su kien da day: {totalFullEvents}</p>
    </div>
  );
};

export default Home;
```

---

### 13. `src/pages/EventList.js`
```jsx
import React, { useState } from 'react';
import FilterBar from '../components/FilterBar';
import EventTable from '../components/EventTable';

const EventList = ({ events, onDelete, onRegister }) => {
  const [search, setSearch] = useState('');
  const [filterStatus, setFilterStatus] = useState('Tat ca');

  const filteredEvents = events.filter((ev) => {
    const matchesSearch = ev.name.toLowerCase().includes(search.toLowerCase());
    const isFull = ev.registered >= ev.capacity;
    let matchesStatus = true;
    if (filterStatus === 'Con cho') {
      matchesStatus = !isFull;
    } else if (filterStatus === 'Da day') {
      matchesStatus = isFull;
    }
    return matchesSearch && matchesStatus;
  });

  return (
    <div>
      <FilterBar
        search={search}
        setSearch={setSearch}
        filterStatus={filterStatus}
        setFilterStatus={setFilterStatus}
      />
      <p>Tim thay {filteredEvents.length} su kien</p>
      {filteredEvents.length === 0 ? (
        <p>Khong co du lieu</p>
      ) : (
        <EventTable events={filteredEvents} onDelete={onDelete} onRegister={onRegister} />
      )}
    </div>
  );
};

export default EventList;
```

---

### 14. `src/pages/AddEvent.js`
```jsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';

const AddEvent = ({ events, onAdd }) => {
  const [name, setName] = useState('');
  const [location, setLocation] = useState('');
  const [date, setDate] = useState('');
  const [capacity, setCapacity] = useState('');
  const [errorMsg, setErrorMsg] = useState('');
  const navigate = useNavigate();

  const handleSubmit = (e) => {
    e.preventDefault();

    if (!name.trim() || !location.trim() || !date || !capacity) {
      setErrorMsg('Vui long nhap dai du thong tin phia duoi form');
      return;
    }

    const capNum = Number(capacity);
    if (isNaN(capNum) || !Number.isInteger(capNum) || capNum < 10 || capNum > 1000) {
      setErrorMsg('Suc chua phai tu 10 den 1000');
      return;
    }

    setErrorMsg('');
    const maxId = events.length > 0 ? Math.max(...events.map((ev) => ev.id)) : 0;
    const newEvent = {
      id: maxId + 1,
      name,
      location,
      date,
      capacity: capNum,
      registered: 0,
    };

    onAdd(newEvent);
    navigate('/events', { replace: true });
  };

  return (
    <div>
      <h2>Them su kien moi</h2>
      <form onSubmit={handleSubmit}>
        <div>
          <label htmlFor="nameInput">Ten su kien: </label>
          <input
            id="nameInput"
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
          />
        </div>
        <div style={{ marginTop: '10px' }}>
          <label htmlFor="locationInput">Dia diem: </label>
          <input
            id="locationInput"
            type="text"
            value={location}
            onChange={(e) => setLocation(e.target.value)}
          />
        </div>
        <div style={{ marginTop: '10px' }}>
          <label htmlFor="dateInput">Ngay dien ra: </label>
          <input
            id="dateInput"
            type="date"
            value={date}
            onChange={(e) => setDate(e.target.value)}
          />
        </div>
        <div style={{ marginTop: '10px' }}>
          <label htmlFor="capacityInput">Suc chua: </label>
          <input
            id="capacityInput"
            type="number"
            value={capacity}
            onChange={(e) => setCapacity(e.target.value)}
          />
        </div>
        <button type="submit" style={{ marginTop: '15px' }}>
          Them
        </button>
      </form>
      {errorMsg && <p className="error-msg">{errorMsg}</p>}
    </div>
  );
};

export default AddEvent;
```

---

### 15. `src/pages/EventDetail.js`
```jsx
import React from 'react';
import { useParams, useNavigate } from 'react-router-dom';
import Countdown from '../components/Countdown';

const EventDetail = ({ events }) => {
  const { id } = useParams();
  const navigate = useNavigate();

  const eventId = Number(id);
  const event = events.find((ev) => ev.id === eventId);

  if (!event || isNaN(eventId)) {
    return <p>Khong tim thay su kien</p>;
  }

  const { name, location, date, capacity, registered } = event;

  return (
    <div>
      <h2>Chi tiet su kien</h2>
      <p>Ten su kien: {name}</p>
      <p>Dia diem: {location}</p>
      <p>Ngay: {date}</p>
      <p>So cho: {registered}/{capacity}</p>
      <p>
        Thoi gian: <Countdown date={date} />
      </p>
      <button onClick={() => navigate(-1)} style={{ marginTop: '15px' }}>
        Quay lai
      </button>
    </div>
  );
};

export default EventDetail;
```