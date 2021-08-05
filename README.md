<h1>스토리북 튜토리얼 따라해보기</h1>
예전에 네이버에서 세미나 같은 것을 할 때, 스토리북에 대한 내용을 접하긴 했는데 직접적으로 사용해 본 적이 없다보니까 이렇게 스스로 튜토리얼을 따라하면서 접해보기로 한다.<br>
영어로 되어있고 내가 한 것은 그저 따라하기 뿐이다.<br>
그래도 내가 공부한 것들을 정리하면서 진행해 볼까 한다.<br>

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
그리고나서 스토리북 빌드 후, 실행시키면 된다.
```shell 
npm run build-storybook
npm run storybook
```
### 4. 상태(State)에 따른 컴포넌트 반영
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
### 8. Composit component
