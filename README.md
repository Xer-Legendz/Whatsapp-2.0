# Whatsapp-2.0
import React, { useEffect, useState, useRef } from "react";

// WhatsApp Web - 1:1 Clone (Single-file React component) // Requirements to run: // 1. Create a new React app (Vite or Create React App). // 2. Install Tailwind CSS and configure it for the project. // 3. Install framer-motion if you'd like animations: npm i framer-motion (optional) // 4. Place this file as WhatsAppClone.jsx and import it into your app root.

// NOTE: This is a functional visual and interactive clone for learning and prototyping only. // Do not copy WhatsApp branding or connect to the real WhatsApp service. Respect copyright // and privacy laws when using or deploying messaging systems.

export default function WhatsAppClone() { // Sample contacts + messages to simulate WhatsApp Web const initialContacts = [ { id: "c1", name: "Ada Lovelace", lastActive: "online", avatarColor: "bg-green-500", messages: [ { id: "m1", fromMe: false, text: "Hey! Are you coming tonight?", time: "09:04" }, { id: "m2", fromMe: true, text: "Yes — I'll be there by 9", time: "09:06" }, ], }, { id: "c2", name: "Alan Turing", lastActive: "Yesterday", avatarColor: "bg-blue-500", messages: [ { id: "m3", fromMe: false, text: "Sent the files — check email.", time: "14:22" }, ], }, { id: "c3", name: "Grace Hopper", lastActive: "2:12", avatarColor: "bg-yellow-500", messages: [ { id: "m4", fromMe: true, text: "Happy birthday! 🎉", time: "07:00" }, ], }, ];

const [contacts, setContacts] = useState(() => { try { const raw = localStorage.getItem("wa_clone_contacts_v1"); return raw ? JSON.parse(raw) : initialContacts; } catch (e) { return initialContacts; } });

const [activeContactId, setActiveContactId] = useState(contacts[0]?.id ?? null); const [query, setQuery] = useState(""); const [input, setInput] = useState(""); const messagesEndRef = useRef(null);

// persist contacts useEffect(() => { localStorage.setItem("wa_clone_contacts_v1", JSON.stringify(contacts)); }, [contacts]);

// scroll to bottom when active contact changes or new message useEffect(() => { scrollToBottom(); }, [activeContactId, contacts]);

function scrollToBottom() { requestAnimationFrame(() => { messagesEndRef.current?.scrollIntoView({ behavior: "smooth" }); }); }

const activeContact = contacts.find((c) => c.id === activeContactId) ?? contacts[0];

function sendMessage(text) { if (!text?.trim() || !activeContact) return; const time = new Date(); const hh = String(time.getHours()).padStart(2, "0"); const mm = String(time.getMinutes()).padStart(2, "0"); const newMsg = { id: "m" + Math.random().toString(36).slice(2, 9), fromMe: true, text, time: ${hh}:${mm} };

setContacts((prev) =>
  prev.map((c) => (c.id === activeContact.id ? { ...c, messages: [...c.messages, newMsg], lastActive: "online" } : c))
);

setInput("");

// fake reply from contact (simulate network / typing)
setTimeout(() => {
  const reply = {
    id: "m" + Math.random().toString(36).slice(2, 9),
    fromMe: false,
    text: generateAutoReply(text),
    time: `${hh}:${String((time.getMinutes() + 1) % 60).padStart(2, "0")}`,
  };
  setContacts((prev) => prev.map((c) => (c.id === activeContact.id ? { ...c, messages: [...c.messages, reply], lastActive: "online" } : c)));
}, 900 + Math.random() * 1400);

}

function generateAutoReply(lastText) { // simple rule-based echo reply to make the UI feel alive if (/hi|hello|hey/i.test(lastText)) return "Hey! 👋"; if (/bye|see you|later/i.test(lastText)) return "Take care!"; if (/thanks|thank you/i.test(lastText)) return "You're welcome 🙂"; return "Nice — got it."; }

function addContact(name) { const id = "c" + Math.random().toString(36).slice(2, 9); const newC = { id, name, lastActive: "Just now", avatarColor: "bg-pink-400", messages: [] }; setContacts((p) => [newC, ...p]); setActiveContactId(id); }

function filteredContacts() { if (!query.trim()) return contacts; const q = query.toLowerCase(); return contacts.filter((c) => c.name.toLowerCase().includes(q) || (c.messages || []).some((m) => m.text.toLowerCase().includes(q))); }

// small UI components inside the same file for simplicity function Avatar({ name, color }) { const initials = name .split(" ") .map((p) => p[0]) .slice(0, 2) .join(""); return ( <div className={w-12 h-12 rounded-full flex items-center justify-center text-white font-semibold ${color}}> {initials} </div> ); }

return ( <div className="h-screen w-screen bg-gray-100 flex text-sm"> {/* Sidebar */} <aside className="w-96 border-r border-gray-200 bg-white flex flex-col"> <header className="px-4 py-3 flex items-center gap-3 border-b"> <div className="flex-1"> <h1 className="text-lg font-semibold">Chat</h1> <p className="text-xs text-gray-500">Prototype — local only</p> </div> <button onClick={() => { const name = prompt("New contact name:"); if (name) addContact(name); }} className="px-3 py-1 rounded-md border text-xs" > + New </button> </header>

<div className="p-3">
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search or start new chat"
        className="w-full rounded-md border p-2 text-sm"
      />
    </div>

    <div className="flex-1 overflow-auto">
      {filteredContacts().map((c) => (
        <button
          key={c.id}
          onClick={() => setActiveContactId(c.id)}
          className={`w-full text-left flex gap-3 items-center px-3 py-2 hover:bg-gray-50 ${c.id === activeContactId ? "bg-gray-50" : ""}`}
        >
          <Avatar name={c.name} color={c.avatarColor} />
          <div className="flex-1">
            <div className="flex items-center justify-between">
              <div className="font-medium">{c.name}</div>
              <div className="text-xs text-gray-400">{c.lastActive}</div>
            </div>
            <div className="text-xs text-gray-500 truncate">
              {c.messages && c.messages.length ? c.messages[c.messages.length - 1].text : "Say hi to start the chat"}
            </div>
          </div>
        </button>
      ))}
    </div>

    <footer className="p-3 border-t text-xs text-gray-500">Local demo — messages saved in your browser only</footer>
  </aside>

  {/* Chat area */}
  <main className="flex-1 flex flex-col">
    {activeContact ? (
      <>
        <div className="flex items-center gap-4 px-6 py-3 border-b bg-white">
          <Avatar name={activeContact.name} color={activeContact.avatarColor} />
          <div className="flex-1">
            <div className="font-semibold">{activeContact.name}</div>
            <div className="text-xs text-gray-500">{activeContact.lastActive}</div>
          </div>
          <div className="flex items-center gap-3">
            <button className="px-3 py-1 rounded-md border text-xs">Call</button>
            <button className="px-3 py-1 rounded-md border text-xs">Menu</button>
          </div>
        </div>

        <div className="flex-1 overflow-auto p-6 bg-[url('data:image/svg+xml;utf8,<svg xmlns=\'http://www.w3.org/2000/svg\' width=\'200\' height=\'200\'><text x=\'0\' y=\'15\' font-size=\'12\' fill=\'%23e5e7eb\'>WhatsApp-style background pattern</text></svg>')]">
          <div className="max-w-3xl mx-auto">
            {(activeContact.messages || []).map((m) => (
              <div key={m.id} className={`mb-3 flex ${m.fromMe ? "justify-end" : "justify-start"}`}>
                <div className={`${m.fromMe ? "bg-green-600 text-white" : "bg-white text-black border"} p-3 rounded-lg shadow-sm max-w-[70%]`}>
                  <div className="whitespace-pre-wrap">{m.text}</div>
                  <div className="text-[10px] text-gray-200 mt-1 text-right">{m.time}</div>
                </div>
              </div>
            ))}
            <div ref={messagesEndRef} />
          </div>
        </div>

        <div className="p-4 border-t bg-white">
          <div className="max-w-3xl mx-auto flex items-center gap-3">
            <button className="p-2 rounded-full border">😊</button>
            <div className="flex-1">
              <input
                value={input}
                onChange={(e) => setInput(e.target.value)}
                onKeyDown={(e) => {
                  if (e.key === "Enter" && !e.shiftKey) {
                    e.preventDefault();
                    sendMessage(input);
                  }
                }}
                placeholder="Type a message"
                className="w-full rounded-full border px-4 py-2 focus:outline-none"
              />
            </div>
            <div className="flex items-center gap-2">
              <label className="px-3 py-2 rounded-md border cursor-pointer text-xs">📎</label>
              <button
                onClick={() => sendMessage(input)}
                className="px-4 py-2 rounded-full bg-green-600 text-white font-semibold"
              >
                Send
              </button>
            </div>
          </div>
        </div>
      </>
    ) : (
      <div className="flex-1 flex items-center justify-center">Select a conversation to start chatting</div>
    )}
  </main>
</div>

); }

