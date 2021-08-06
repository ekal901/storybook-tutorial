<h1>스토리북 튜토리얼 따라해보기</h1>
예전에 네이버에서 세미나 같은 것을 할 때, 스토리북에 대한 내용을 접하긴 했는데 직접적으로 사용해 본 적이 없다보니까 이렇게 스스로 튜토리얼을 따라하면서 접해보기로 한다.<br>
영어로 되어있고 내가 한 것은 그저 따라하기 뿐이다. 그래도 내가 공부한 것들을 정리하면서 진행해 볼까 한다.<br>

### 1. 프로젝트 만들기
```shell
npx degit chromaui/intro-storybook-react-template taskbox 
cd taskbox
npm install
```
### 2. 컴포넌트 만들기
<img alt="component" src="https://storybook.js.org/tutorials/intro-to-storybook/task-states-learnstorybook.png" width="100%" />
이 태스크 UI를 Component-Driven Development(CDD) 방식으로 만들거라고 한다.<br>
뭔가 내용이 많지만 우선 컴포넌트를 만들어보기로 한다.<br>
<br>

```javascript <br>
// src/components/Task.js
import React from 'react';
export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```
```javascript <br>
//src/components/Task.stories.js
import React from "react";

import Task from './Task';

export default {
    component: Task,
    title: 'Task'
};

const Template = args => <Task {...args} />;

export const Default = Template.bind({});
Default.args = {
    task: {
        id: '1',
        title: 'Test Task',
        state: 'TASK_INBOX',
        updatedAt: new Date()
    }
}

export const Pinned = Template.bind({});
Pinned.args = {
    task: {
        ...Default.args.task,
        state: 'TASK_PINNED'
    }
}

export const Archived = Template.bind({});
Archived.args = {
    task: {
        ...Default.args.task,
        state: 'TASK_ARCHIVED'
    }
}
```
Task.js와 Task.stories.js 파일을 만들어 준다.<br>
잘 보면, bind를 사용하여, 각 상태별 컴포넌트를 만들어주는것을 볼 수 있다.
Default.args.task의 state를 덮어쓴다. (나중에 선언됨)

### 3. Config 파일 수정하기
```javascript
// taskbox/.storybook>main.js

module.exports = {
  "stories": [
    "../src/components/**/*.stories.js"
  ],
  "addons": [
    "@storybook/addon-links",
    "@storybook/addon-essentials",
    "@storybook/preset-create-react-app"
  ]
}
```
```javascript
//taskbox/.storybook>preview.js
import '../src/index.css'

export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
//   controls: {
//     matchers: {
//       color: /(background|color)$/i,
//       date: /Date$/,
//     },
//   },
};
```

### 중요! 
```shell 
npm run build-storybook
npm run storybook
```
스토리북 빌드 후, 실행을 시켜야 에러가 나지 않는다. 문서에서는 이것에 대해 설명해 두지 않아 조금 헤맸다.
### 4. 상태(State)에 따른 컴포넌트 반영
태스크 완료라면, 체크박스를 disable 처리 및 list-item.TASK_ARCHIVED input이 disable 처리되어 보임
```javascript
// src/components/Task.js
import React from 'react';
import PropTypes from 'prop-types';

export default function Task({ task : { id, title, state}, onArchiveTask, onPinTask}) {
    return (
        <div className={`list-item ${state}`}>
            <label className="checkbox">
                <input 
                    type="checkbox" 
                    defaultChecked={state === 'TASK_ARCHIVED'} 
                    disabled={true} 
                    name="checked" 
                />
                <span className="checkbox-custom" onClick={() => onArchiveTask(id)} />
            </label>

            <div className="title">
                <input type="text" value={title} readOnly={true} placeholder="Input title" />
            </div>

            <div className="actions" onClick={event => event.stopPropagation()}>
                {state !== 'TASK_ARCHIVED' && (
                    // eslint-disable-next-line jsx-a11y/anchor-is-valid
                    <a onClick={() => onPinTask(id)}>
                        <span className={`icon-star`} />
                    </a>
                )}
            </div>
        </div>
    )
}
```

### 5. 필수 데이터값 정하기 (PropTypes 사용)
PropType 대신에 Typescript를 사용하는 것도 괜찮을 것 같다.
```javascript
// src/components/Task.js
Task.propTypes = {
    task: PropTypes.shape({
        id: PropTypes.string.isRequired,
        title: PropTypes.string.isRequired
    }),
    onArchiveTask: PropTypes.func,
    onPinTask: PropTypes.func
}
```
### 6. 스냅샷 테스팅
**`스탭샷 테스팅`** : UI가 예상 밖으로 변경되지 않도록 하기 위할때 쓴다. 난 아직 활용이나 이런것에 익숙하지 않아서 조금 어렵다. 캡처해서 비교한다 이런 느낌으로 이해했다.

```shell
npm install -D @storybook/addon-storyshots
npm install -D react-test-renderer
```
```javascript
// src/storybook.test.js
import initStoryshots from '@storybook/addon-storyshots';
initStoryshots();
```
## Composit component
이번엔 좀 더 복잡한 리스트를 만든다고 한다.<br> Pinned 될 때는 Default 보다 위에 있는 리스트 하나, 그리고 Default 들만 있는 리스트 하나<br>
<img alt="composit" src="https://storybook.js.org/tutorials/intro-to-storybook/tasklist-states-1.png" width="100%">
그리고 로딩 안될 때 화면 하나, 그리고 Task가 없을때 화면도 추가한다고 한다.<br>
### 1. Get Setup
```javascript
// src/components/TaskList.js 
import React from 'react';

import Task from './Task';

export default function TaskList({ loading, tasks, onPinTask, onArchiveTask}) {
    const events = {
        onPinTask,
        onArchiveTask
    };

    if(loading) {
        return <div className="list-items">loading</div>;
    }

    if(tasks.length === 0) {
        return <div className="list-items">empty</div>
    }

    return (
        <div className="list-items">
            // map을 사용해서 Task 리스트 뿌려주기, Task에서는 개별 값을 가지고 div 그린다.
            {tasks.map(task => (
                <Task key={task.id} task={task} {...events}></Task>
            ))}
        </div>
    )
}
```
### 2. TaskList의 stories 파일 만들기
```javascript
// src/components/TaskList.stories.js
import React from 'react';

import TaskList from './TaskList';
import * as TaskStories from './Task.stories';

export default {
    component: TaskList,
    title: 'TaskList',
    decorators: [story => <div style={{padding: '3rem'}}>{story()}</div>]
};

const Template = args => <TaskList {...args} />

export const Default = Template.bind({});
Default.args = {
    tasks: [
        {...TaskStories.Default.args.tasks, id: '1', title: 'Task 1'},
        {...TaskStories.Default.args.tasks, id: '2', title: 'Task 2'},
        {...TaskStories.Default.args.tasks, id: '3', title: 'Task 3'},
        {...TaskStories.Default.args.tasks, id: '4', title: 'Task 4'},
        {...TaskStories.Default.args.tasks, id: '5', title: 'Task 5'},
        {...TaskStories.Default.args.tasks, id: '6', title: 'Task 6'}
    ]
};

export const WithPinnedTask = Template.bind({});
WithPinnedTask.args = {
    tasks: [
        ...Default.args.tasks.slice(0,5), // 객체 5개 slice
        {id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED'}
    ]
}

export const Loading = Template.bind({});
Loading.args = {
    tasks: [],
    loading: true
};

export const Empty = Template.bind({});
Empty.args = {
    tasks: [],
    loading: false
}
```

### 3. States 에 따른 UI를 TaskList.js에 반영
기존의 단순한 UI 구조를 TaskList.js 파일을 수정해서 좀 더 디테일하게 다듬었다.
TaskList.js에 아래 코드를 추가한다.
```javascript
// src/components/TaskList.js
import React from 'react';

import Task from './Task';

export default function TaskList({ loading, tasks, onPinTask, onArchiveTask}) {
    const events = {
        onPinTask,
        onArchiveTask
    };

    const LoadingRow = (
        <div className="loading-item">
            <span className="glow-checkbox" />
            <span className="glow-text">
                <span>Loading</span> <span>cool</span> <span>cool</span>
            </span>
        </div>
    )

    if(loading) {
        return <div className="list-items">
            {LoadingRow}
            {LoadingRow}
            {LoadingRow}
            {LoadingRow}
            {LoadingRow}
            {LoadingRow}
        </div>;
    }

    if(tasks.length === 0) {
        return (
            <div className="list-items">
                <div className="wrapper-message">
                    <span className="icon-check" />
                    <div className="title-message">You have no tasks</div>
                    <div className="subtitle-message">Sit back and relax</div>
                </div>
            </div>
        )
    }

    const taskInOrder = [
        ...tasks.filter(t => t.state === 'TASK_PINNED'), // 필터를 사용해서 PINNED 된 것들은 상단에 위치
        ...tasks.filter(t => t.state !== 'TASK_PINNED'), // 나머지는 하단에 위치
    ]

    return (
        <div className="list-items">
            {taskInOrder.map(task => (
                <Task key={task.id} task={task} {...events}></Task>
            ))}
        </div>
    )
}
```
https://storybook.js.org/tutorials/intro-to-storybook/finished-tasklist-states-6-0.mp4<br>
동작 이미지는 위에 링크에서 확인 가능하다.

### 4. propTypes 추가
TaskList.js 파일의 하단에 propTypes에 대한 정의를 추가
```javascript
import PropTypes from 'prop-types'; // 상단에 추가!

TaskList.propTypes = {
    loading: PropTypes.bool,
    tasks: PropTypes.arrayOf(Task.propTypes.task).isRequired,
    onPinTask: PropTypes.func,
    onArchiveTask: PropTypes.func
};

TaskList.defaultProp = {
    loading: false
};
```
### 5. 자동화 테스트 
React Testing Library와 @storybook/testing-react를 사용<br>
```shell
```
