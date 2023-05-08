---
title: State ম্যানেজমেন্ট
---

<Intro>

আপনি আপনার এপ্লিকেশনে কিভাবে State গুলো কে সাজাবেন এবং এপ্লিকেশনের ডেটা কিভাবে এক কম্পনেন্ট থেকে অন্য কম্পনেন্টে পাঠাবেন, এই বিষয়গুলোর প্রয়োজনীয়তা এপ্লিকেশন বড় হওয়ার সাথে সাথে বৃদ্ধি পায়। যেখানে অপ্রয়োজনীয় অথবা একই নামের একাধিক State এপ্লিকেশনের একটা সাধারণ বাগ। এই চ্যাপ্টার আপনি শিখবেন কিভাবে সঠিকভাবে State সাজাতে হয়, কিভাবে State আপডেট এর লজিক লেখা যায় যেন তা পরবর্তীতে যেকোনো অবস্থায় ব্যবহার করা সহজ হয়, এবং কিভাবে এক কম্পনেন্ট এর State অন্য কোনো দূরবর্তী কম্পনেন্টে পাঠানো যায়।

</Intro>

<YouWillLearn isChapter={true}>

* [State পরিবর্তনের সাথে সাথে UI পরিবর্তনগুলি সম্পর্কে কীভাবে ভাববেন](/learn/reacting-to-input-with-state)
* [কিভাবে State গুলোকে সঠিকভাবে সাজাবেন](/learn/choosing-the-state-structure)
* [কম্পনেন্ট এর মধ্যে State শেয়ার করার জন্য কীভাবে State গুলোকে তৈরি করবেন করবেন](/learn/sharing-state-between-components)
* [State সংরক্ষিত বা পুনঃস্থাপিত হয় কিনা তা কিভাবে নিয়ন্ত্রণ করবেন](/learn/preserving-and-resetting-state)
* [একটি ফাংশনে জটিল State লজিক কীভাবে একত্রিত করবেন](/learn/extracting-state-logic-into-a-reducer)
* [Props Driling ছাড়া কিভাবে তথ্য আদান প্রদান করবেন](/learn/passing-data-deeply-with-context)
* [এপ্লিকেশন বড় হওয়ার সাথে সাথে কিভাবে আপনার State গুলো কে স্কেল করবেন](/learn/scaling-up-with-reducer-and-context)

</YouWillLearn>

## State দ্বারা ইনপুট এর প্রতিক্রিয়া {/*reacting-to-input-with-state*/}

React এর মাধ্যমে, আপনি কোডের মাধ্যমে সরাসরি কখনও UI পরিবর্তন করবেন না। যেমন, আপনি কখনও এভাবে লিখবেন না যে, "disable the button", "enable the button", "show the success message", ইত্যাদি। এর পরিবর্তে আপনি এটা আপনার UI গুলো ডিফাইন করে রাখবেন যেগুলো আপনি কম্পনেন্ট এর ভিন্ন ভিন্ন দৃশ্যমান State এ দেখতে চান ("initial state", "typing state", "success state"), এবং ইউজার ইনপুটের প্রতিক্রিয়া হিসাবে Stateের পরিবর্তনগুলি ট্রিগার করবেন। ডিজাইনার রা যেভাবে UI ডিজাইন করার সময় চিন্তা করে এটিও ঠিক সেরকমই।

নিচে একটি React দিয়ে বানানো কুইজ ফর্মের কোড দেয় হলো। লক্ষ করুন এটা কিভাবে `status` নামক State টি ব্যবহার করে কিভাবে সাবমিট বাটন টি এনেইবল অথবা ডিজেইবল করছে, এবং এছড়া কখন একটি সাকসেস মেসেজ দেখাচ্ছে।

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>সঠিক উত্তর!</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>জেলা সম্পর্কিত কুইজ</h2>
      <p>
        বর্তমানে কোন জেলায় বাংলাদেশের প্রথম অস্থায়ী রাজধানী গঠিত হয়?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          Submit
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // মনে করুন এটি একটি নেটওয়ার্ক হিট করছে
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'meherpur' && answer !== 'মেহেরপুর'
      if (shouldError) {
        reject(new Error('উত্তর সঠিক নয়, আবার চেষ্টা করুণ!'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

```css
.Error { color: red; }
```

</Sandpack>

<LearnMore path="/learn/reacting-to-input-with-state">

State দ্বারা UI এর সাথে interactions সম্পর্কে আরও জানতে **[State দ্বারা ইনপুট এর প্রতিক্রিয়া](/learn/reacting-to-input-with-state)** পেইজটি পড়ুন।

</LearnMore>

## State এর সঠিক কাঠামো নির্বাচন {/*choosing-the-state-structure*/}

State কে সুন্দরভাবে সাজানোর মাধ্যমে একটি কম্পনেন্ট যা পরিবর্তন এবং ডিবাগ করা সহজ এবং একটি কম্পনেন্ট যা বাগ দ্বারা পরিপূর্ণ। এক্ষেত্রে সবথেকে গুরুত্বপূর্ণ নীতি হচ্ছে State এ অপ্রয়োজনীয় বা সদৃশ তথ্য থাকা উচিত নয়। যদি কোনো অপ্রয়োজনীয় State থাকে, পরবর্তীতে তা আপডেট করতে সাধারণত ভুল হয়ে যায়, এবং এর থেকে ব্যাগ এর উৎপত্তি হয়ে থাকে।

উদাহরণস্বরূপ, নিচের ফর্মে  `fullName` নামক একটি **অপ্রয়োজনীয়** State আছে।

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>চেক ইন করুণ</h2>
      <label>
        নামের প্রথম অংশ:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        নামের শেষাংশ:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        আপনার টিকিট টি <b>{fullName}</b> নামে বুকিং করা হয়েছে।
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

আপনি এটিকে রিমুভ করতে পারেন এবং কম্পোনেন্টটি রেন্ডার করার সময় `fullName` একত্র করে কোডটিকে আরও সহজ করতে পারেন:

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>চেক ইন করুণ</h2>
      <label>
        নামের প্রথম অংশ:{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        নামের শেষাংশ:{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
         আপনার টিকিট টি <b>{fullName}</b> নামে বুকিং করা হয়েছে।
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

এটি একটি ছোট পরিবর্তনের মতো মনে হতে পারে, তবে React অ্যাপে অনেক বাগ এইভাবে ঠিক করা হয়ে থাকে।

<LearnMore path="/learn/choosing-the-state-structure">

বাগ এড়ানোর জন্য কিভাবে State ডিজাইন করবেন এ সম্পর্কে জানতে **[State এর সঠিক কাঠামো নির্বাচন](/learn/choosing-the-state-structure)** পেইজটি পড়ুন।

</LearnMore>

## বিভিন্ন কম্পনেন্টের মধ্যে State আদান প্রদান করা {/*sharing-state-between-components*/}

মাঝে মাঝে আপনার একাধিক কম্পনেন্টের State একই সাথে পরিবর্তন করার প্রয়োজন হবে, সেক্ষেত্রে আপনি উভয় কম্পনেন্ট থেকেই State রিমুভ করে দিয়ে, সেই State কে তাদের সবথেকে কাছের parent কম্পনেন্টে রাখুন এবং Props এর মাধ্যমে তা উভয় কম্পনেন্টে পাঠান। এই প্রক্রিয়া টি "lifting state up" নামে পরিচিত, এবং এটি আপনার React এর সবথেকে বেশিবার করা কাজের মধ্যে একটি হবে।

এই উদাহরণে, একই সাথে শুধুমাত্র একটি প্যানেল চালু থাকবে। এটি করতে হলে, একটিভ State টি কে প্রতিটি প্যানেল এর মধ্যে না রেখে তাঁকে parent কম্পনেন্টে রাখা হবে এবং child কম্পনেন্ট গুলোর জন্য Props উল্লেখ করে দিতে হবে।

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>ঢাকা, বাংলাদেশ</h2>
      <Panel
        title="পরিচিতি"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        ঢাকা দক্ষিণ এশিয়ার একটি গুরুত্বপূর্ণ রাষ্ট্র বাংলাদেশের রাজধানী ও বৃহত্তম শহর। প্রশাসনিকভাবে এটি ঢাকা বিভাগের ও জেলার প্রধান শহর। ঢাকা মহানগরীর মোট জনসংখ্যা প্রায় ২ কোটি ১০ লক্ষ, যা দেশের মোট জনসংখ্যার প্রায় ১১ ভাগ। জনঘনত্বের বিচারে ঢাকা বিশ্বের সবচেয়ে ঘনবসতিপূর্ণ শহর; ৩০৬ বর্গকিলোমিটার আয়তনের এই শহরে প্রতি বর্গকিলোমিটার এলাকায় ২৩ হাজার লোক বাস করে।
      </Panel>
      <Panel
        title="নামকরণের ইতিহাস"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        ঢাকার নামকরণের সঠিক ইতিহাস নিয়ে ব্যাপক মতভেদ রয়েছে। কথিত আছে যে, সেন বংশের রাজা বল্লাল সেন বুড়িগঙ্গা নদীর তীরবর্তী এলাকায় ভ্রমণকালে সন্নিহিত জঙ্গলে হিন্দু দেবী দুর্গার একটি বিগ্রহ খুঁজে পান। দেবী দুর্গার প্রতি শ্রদ্ধাস্বরূপ রাজা বল্লাল সেন ঐ এলাকায় একটি মন্দির প্রতিষ্ঠা করেন। যেহেতু দেবীর বিগ্রহ ঢাকা বা গুপ্ত অবস্থায় খুঁজে পাওয়া গিয়েছিলো, তাই রাজা, মন্দিরের নাম রাখেন ঢাকেশ্বরী মন্দির। মন্দিরের নাম থেকেই কালক্রমে স্থানটির নাম ঢাকা হিসেবে গড়ে ওঠে।

        আবার অনেক ঐতিহাসিকের মতে, মোঘল সম্রাট জাহাঙ্গীর যখন ঢাকাকে সুবাহ বাংলার রাজধানী হিসেবে ঘোষণা করেন; তখন সুবাদার ইসলাম খান আনন্দের বহিঃপ্রকাশস্বরূপ শহরে “ঢাক” বাজানোর নির্দেশ দেন। এই ঢাক বাজানোর কাহিনী লোকমুখে কিংবদন্তির রূপ নেয় এবং তা থেকেই শহরের নাম ঢাকা হয়ে যায়। এখানে উল্লেখ্য যে, মোঘল সাম্রাজ্যের বেশ কিছু সময় ঢাকা সম্রাট জাহাঙ্গীরের প্রতি সম্মান জানিয়ে জাহাঙ্গীরনগর নামে পরিচিত ছিলো।
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          Show
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<LearnMore path="/learn/sharing-state-between-components">

State এবং Props মাধ্যমে কিভাবে একাধিক কম্পনেন্ট সিঙ্কে রাখতে হয় তা **[বিভিন্ন কম্পনেন্টের মধ্যে State আদান প্রদান করা](/learn/sharing-state-between-components)** পেইজটি পড়ুন।

</LearnMore>

## State সংরক্ষণ এবং রিসেট করা {/*preserving-and-resetting-state*/}

যখন আপনি একটি কম্পনেন্ট পুনরায় render করবেন, সেখানে কম্পনেন্ট ট্রি এর কোন অংশটিকে রেখে দিতে হবে (আপডেট করতে হবে), এবং কোন অংশটিকে মুছে দিয়ে পুনরায় শুরু থেকে তৈরি করতে হবে। বেশিরভাগ ক্ষেত্রেই React নিজেই এটি ভালোভাবে করে থাকে। React কম্পনেন্ট ট্রি এর সেই অংশ গুলিকে সংরক্ষণ করে রাখে যা পূর্বে render হওয়া কম্পনেন্ট ট্রি এর সাথে মিলে যায়।

যায়হোক, মাঝে মাঝে এটি হোক আপনি চান না। এই চ্যাট অ্যাপ টি তে, একটি মেসেজ লেখার পর প্রাপক পরিবর্তন করলে তা ইনপুট কে রিসেট করে না। যার ফলে ভুল ইউজার কে ভুল মেসেজ পাঠিয়ে দেয়ার সম্ভাবনা থাকে:

<Sandpack>

```js App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'রহিম', email: 'rahim@mail.com' },
  { name: 'করিম', email: 'karim@mail.com' },
  { name: 'রীতা', email: 'rita@mail.com' }
];
```

```js ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={contact.name + " কে মেসেজ লিখুন"}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>{contact.email} এড্রেসে পাঠান</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>


React আপনাকে এর ডিফল্ট আচরণ পরিবর্তন করতে দেয়, এবং একটি ভিন্ন *key* পাঠানোর মাধ্যমে জোরপূর্বক একটি কম্পনেন্টের State রিসেট করতে দেয়। যেমন `<Chat key={email} />`। এটি React কে বলে যে প্রাপক ভিন্ন হলে, এটি একটি *ভিন্ন* `চ্যাট` উপাদান হিসাবে বিবেচিত হবে যা নতুন ডেটা (এবং ইনপুটের মতো UI) দিয়ে শুরু থেকে পুনরায় তৈরি করা দরকার। এখন প্রাপকদের মধ্যে স্যুইচ করা ইনপুট field পুনরায় সেট করে যদিও আপনি একই কম্পনেন্ট রেন্ডার করেন।

<Sandpack>

```js App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.email} contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'রহিম', email: 'rahim@mail.com' },
  { name: 'করিম', email: 'karim@mail.com' },
  { name: 'রীতা', email: 'rita@mail.com' }
];
```

```js ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={contact.name + " কে মেসেজ লিখুন"}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>{contact.email} এড্রেসে পাঠান</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<LearnMore path="/learn/preserving-and-resetting-state">

State এর জীবনকাল এবং কিভাবে এটি নিয়ন্ত্রণ করতে হয় তা শিখতে **[State সংরক্ষণ এবং রিসেট করা](/learn/preserving-and-resetting-state)** পেইজটি পড়ুন।

</LearnMore>

## Extracting state logic into a reducer {/*extracting-state-logic-into-a-reducer*/}

Components with many state updates spread across many event handlers can get overwhelming. For these cases, you can consolidate all the state update logic outside your component in a single function, called "reducer". Your event handlers become concise because they only specify the user "actions". At the bottom of the file, the reducer function specifies how the state should update in response to each action!

<Sandpack>

```js App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false }
];
```

```js AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Add</button>
    </>
  )
}
```

```js TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<LearnMore path="/learn/extracting-state-logic-into-a-reducer">

Read **[Extracting State Logic into a Reducer](/learn/extracting-state-logic-into-a-reducer)** to learn how to consolidate logic in the reducer function.

</LearnMore>

## Passing data deeply with context {/*passing-data-deeply-with-context*/}

Usually, you will pass information from a parent component to a child component via Props. But passing Props can become inconvenient if you need to pass some prop through many components, or if many components need the same information. Context lets the parent component make some information available to any component in the tree below it—no matter how deep it is—without passing it explicitly through Props.

Here, the `Heading` component determines its heading level by "asking" the closest `Section` for its level. Each `Section` tracks its own level by asking the parent `Section` and adding one to it. Every `Section` provides information to all components below it without passing Props--it does that through context.

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

```js Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

```js LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<LearnMore path="/learn/passing-data-deeply-with-context">

Read **[Passing Data Deeply with Context](/learn/passing-data-deeply-with-context)** to learn about using context as an alternative to passing Props.

</LearnMore>

## Scaling up with reducer and context {/*scaling-up-with-reducer-and-context*/}

Reducers let you consolidate a component’s state update logic. Context lets you pass information deep down to other components. You can combine reducers and context together to manage state of a complex screen.

With this approach, a parent component with complex state manages it with a reducer. Other components anywhere deep in the tree can read its state via context. They can also dispatch actions to update that state.

<Sandpack>

```js App.js
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```js TasksContext.js
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider
        value={dispatch}
      >
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

```js AddTask.js
import { useState, useContext } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        });
      }}>Add</button>
    </>
  );
}

let nextId = 3;
```

```js TaskList.js
import { useState, useContext } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<LearnMore path="/learn/scaling-up-with-reducer-and-context">

Read **[Scaling Up with Reducer and Context](/learn/scaling-up-with-reducer-and-context)** to learn how state management scales in a growing app.

</LearnMore>

## What's next? {/*whats-next*/}

Head over to [Reacting to Input with State](/learn/reacting-to-input-with-state) to start reading this chapter page by page!

Or, if you're already familiar with these topics, why not read about [Escape Hatches](/learn/escape-hatches)?
