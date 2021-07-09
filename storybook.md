# storybook初探：利用storybook构建组件文档库 

Storybook常用来打造团队的UI组件库，它可以对各个组件进行测试与编写文档。Storybook可以用于React，Vue和Angular上，本文主要介绍Storybook + React系统的构建。

本文将通过以下的顺序来介绍该系统的构建：

* 构建一个基本的react + storybook框架

* 完成一个基础的UI组件库

* 对其进行进一步的编辑与部署

## 创建一个React + Storybook 项目

```javascript
// 创建react项目
npx create-react-app story

cd story

// 添加storybook
npx -p @storybook/cli sb init
```

在执行完以下语句后，我们查看项目的目录。可以发现，相较于一般react项目，storybook添加了一些文件。

```
├── .storybook
│       │── main.js         //主文件
│       └── preview.js      //预览设置文件
│
├─ src 
│    └── stories
│             │── assets              //用于存放静态资源的文件夹
│             │── button.css          //按钮组件样式文件
│             │── Button.jsx          //按钮组件主文件
│             │── Button.stories.jsx  //按钮组件文档文件
│             │── Introduction.stories.mdx //欢迎介绍页面
│             └── .....
```

其中，`.storybook`文件用于配置一些storybook的整体配置。cli脚手架已自动配置好了`main.js`和`preview.js`。

同时，storybook模板还为我们提供了一些组件样例，这些组件的样例都存放在`src/stories`中，统一命名为`xxx.stories.js`

为了区分组件与stories文档，我们一般会对组件的目录进行修改，将组件主文件和文档文件分开存放。

```
├── .storybook
│       │── main.js         //主文件
│       └── preview.js      //预览设置文件
│
├─ src 
│    └── stories
│             │── assets              //用于存放静态资源的文件夹
│             │── Button.stories.jsx  //按钮组件文档文件
│             │── Introduction.stories.mdx //欢迎介绍页面
│             └── .....
│    └── components 
│             │── button.css          //按钮组件样式文件
│             │── Button.jsx          //按钮组件主文件  
│             └── .....
```

### 1. main.js

`.storybook`中的`main.js`文件主要用于读取组件和接口，它可以设置storybook读取哪个文件夹的内容，匹配文件夹中的哪些文件。

```javascript
const path = require('path');

module.exports = {

  // 配置引入组件的路径
  stories: [
    "../src/**/*.stories.mdx",
    "../src/**/*.stories.@(js|jsx|ts|tsx)"
  ],

  // 添加需要引入的组件
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    "@storybook/preset-create-react-app"
  ],
};
```

main文件还用于描述引入的插件，可以在此处添加一些你需要的插件，例如"@storybook/addon-docs"。在storybook的官网可以找到所有的插件 ==>[addons插件](https://storybook.js.org/addons)

### 2. preview.js

`preview.js`主要用于进行一些全局设置以及自定义左侧导航栏的排列顺序。这个文件可以配置全局（即所有组件）的显示属性

```javascript
export const parameters = {
  actions: { argTypesRegex: "^on[A-Z].*" },
  backgrounds: {
    default: 'light'
  },
  layout: 'centered',
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
    expanded: true
  }
}
```

例如，这里设置默认主题为`light`，组件的显示方式为居中，控制栏自定义了color和date的匹配规则，以及设置为控制栏显示全部属性，文档部位内联布局。

此时，在完成基础的配置后，运行`yarn storybook`即可预览出基础的组件库。

## 组件库文档添加组件

接下来，可以对已有的组件进行编辑，添加组件文档。组件文档有两种编写方式，Component Story Format(CSF) 和 MDX。CSF采用JSX的方式编辑，而MDX则是通过Markdown与jsx结合的方式来编写文档，这两者都存在各自的优势。

我们先以默认的CSF（即JSX）编写方式来展开。首先，我们完成一个组件的构建，以按钮Button组件为例，我们基于react构建了一个button按钮，同时为组件添加prop-types包，定义组件每个属性的propTypes后，storybook可以读取组件的属性，生成controls控制台。同时，也会接收属性的defaultProps（默认值）。

```javascript
import React from 'react';
import PropTypes from 'prop-types';
import './button.css';

export const Button = ({ primary, backgroundColor, size, label, ...props }) => {
  const mode = primary ? 'storybook-button--primary' : 'storybook-button--secondary';
  return (
    <button
      type="button"
      className={['storybook-button', `storybook-button--${size}`, mode].join(' ')}
      style={backgroundColor && { backgroundColor }}
      {...props}
    >
      {label}
    </button>
  );
};

Button.propTypes = {
  primary: PropTypes.bool,
  backgroundColor: PropTypes.string,
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  label: PropTypes.string.isRequired,
  onClick: PropTypes.func,
};

Button.defaultProps = {
  backgroundColor: null,
  primary: false,
  size: 'medium',
  onClick: undefined,
};

```

构造完成组件之后，可以在story文件中导入该组件，为该组件编写组件文档。组件文档文件命名为Button(组件名).stories.js，通过编写该文件，可以生成文档。
一个最基础的组件文档有以下几个要素：

1. 标题：文档的标题

2. 组件：文档引入并描述的是哪一个组件，并将组件展示在文档中

3. 控制栏：列举了组件的各个属性，并且可以通过操作这些属性来改变组件的样式

根据这些需求，我们可以生成一个最基础的组件文档。

```javascript
import React from 'react';

import { Button } from './Button';

export default {

  // 组件的标题：分类/组件名
  title: 'Example/Button',

  // 使用的组件
  component: Button,

  // 此处可以添加控制栏中的属性，其他的属性已通过propTypes自动导入
  argTypes: {
    backgroundColor: { control: 'color' },
  },
};

// 创建一个组件，并关联属性
const Template = (args) => <Button {...args} />;

// 将组件暴露在文档中，并添加一些默认属性
export const Primary = Template.bind({});
Primary.args = {
  primary: true,
  label: 'Button',
};

```

此时运行`yarn storybook`命令，则可以生成一个基础的组件文档。

![基础文档](https://github.com/Dasiylight/note/blob/master/picture/basic.png?raw=true)

同时，我们可能需要同时展示一个组件的多个形态，以展示组件的属性。因此我们可以在文档中创建多个具有不同属性的组件

```javascript
const Template = (args) => <Button {...args} />;

export const Primary = Template.bind({});
Primary.args = {
  primary: true,
  label: 'Button',
};

export const Secondary = Template.bind({});
Secondary.args = {
  label: 'Button',
};

export const Large = Template.bind({});
Large.args = {
  size: 'large',
  label: 'Button',
};

export const Small = Template.bind({});
Small.args = {
  size: 'small',
  label: 'Button',
};
```

上述代码通过`Template.bind({})`方式复制了多个组件，每一个这样的组件称为一个**story**（故事）。一个component可以生成多个story，这些story可以在docs文档中被调用。

## 进一步地优化组件文档

在生成了一个基础文档后，可以进一步对文档进行编辑来更好的展示组件。可以从以下几点来对文档进行优化。

1. 优化控制栏，并按需引入副组件

2. 增加描述，并修改文档的结构

3. 采用mdx编写文档

### 控制栏优化

首先，在生成文档后，可以观察文档的结构，由canvas与docs构成。canvas只用于展示组件，并对组件进行操作，且控制台默认只展示Name(组件名)与Control(属性值)。可以通过在`preview.js`中设置显示全部属性。

```javascript
// preview.js

export const parameters = {
  controls: {
    expanded: true
  },
}
```

此时，我们可以看到控制栏中Description一栏中仅有属性的类型。因此优化组件的第一步就是对组件的类型进行优化。我们可以为组件的每个属性添加描述，让人可以更好的了解组件每个属性起到的作用。同时，虽然Control中已填入通过defaultProps导入的值，Default栏仍显示为空。

```javascript
// Alert.stories.js

import React from 'react';
import { Alert } from './Alert';

export default {
  component: Alert,
  title: "展示/Alert(提示)",
  argTypes: {
    text: {
      description: "简短的描述",
      defaultValue: { summary: "用于给用户显示提示信息。" },
      table: {
        type: {
          summary: "string",
          detail: "这里是点击类型展开之后的详细描述"
        },
      }
    },
    type: {
      description: "类型",
      defaultValue: { summary: "success" },
      type: {
        options: ["success", "warning", "default", "error"],
        type: 'radio'
      }
    }
  }
};

```

这时我们可以看到已经为`text`和`type`两个属性的Default栏添加了默认值标签。

![默认值显示](https://github.com/Dasiylight/note/blob/master/picture/defaultValue.png?raw=true)

除此之外，还可以通过在组件主文件中添加注释的方式，来直接对组件的各个参数添加描述。在组件的propTypes属性中，在每个参数上添加注释，组件文档即可自动导入描述

```javascript
// Button.jsx

Button.propTypes = {
  /**
   * Is this the principal call to action on the page?
   */
  primary: PropTypes.bool,
  /**
   * What background color to use
   */
  backgroundColor: PropTypes.string,
  /**
   * How large should the button be?
   */
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  /**
   * Button contents
   */
  label: PropTypes.string.isRequired,
  /**
   * Optional click handler
   */
  onClick: PropTypes.func,
};

```

### 文档描述优化

默认生成的基础docs文档仅有标题、组件栏与控制栏三个内容。作为一个文档显然需要对组件进行描述。我们可以为文档添加：副标题、正文内容以及各式各样的story，以此来丰富文档内容。  

首先，当需要子组件来对组件进行辅助描述时，可以利用subcomponents将子组件的属性一同列举出来。  

例如：`Form`和`FormItem`

```javascript
// ButtonGroup.stories.js

import { Form } from '../Form';
import { FormItem } from '../FormItem';


export default {
  title: 'Path/to/Form',
  component: Form,
  subcomponents: { FormItem },
};  
```

Docs就会将子组件的参数显示在主要组件旁。

接下来我们可以自定义Docs页面，可以使用Parameters来进行覆写。一个Docs页面可以拆分为许多Docs Blocks，我们自定义Docs页面，就是将构建出一个个Docs Blocks，并将其进行排列组合。

以一个demo为例：

```javascript
import React from 'react';
import Alert from '../../web/components/Alert';

import {
  Title,
  Subtitle,
  Description,
  Primary,
  ArgsTable,
  Stories,
  PRIMARY_STORY,
} from '@storybook/addon-docs/blocks';

export default {
  component: Alert,
  title: "展示/提示(Alert)",
  parameters: {
    docs: {
      description: {
        component: '组件描述',
      },
      page: () => (
        <>
          <Title />
          <Subtitle />
          <Description />
          <Primary />
          <ArgsTable story={PRIMARY_STORY} />
          <Stories />
        </>
      ),
    },
    componentSubtitle: "副标题"
  },
};

```

可以看到，通过在page中引入各个block，以达到自定义的目的。以下是可以设置的Docs Blocks，如果有需要的话可以自行排列block的顺序。

* `<Title />`
  
用于显示文档的标题，可以在title中定义

```javascript
title: '展示/Alert(提示)'
```

也可以通过Slot的方式传入

```javascript
<Title>标题</Title>
```

* `<Subtitle />`

默认传入parameters.componentSubtitle中的值，也可以通过Slot的方式传入

* `<Description />`

默认传入parameters.docs.description.component|story中的文字，也可以通过Slot的方式传入

* `<Primary />`

显示第一个（Primary）设定的story

* `<ArgsTable />`

显示Primary Stories传入的参数列表

* `<Stories />`

显示除Primary Story之外的stories，可以添加title来设定此区块的标题

```javascript
<Stories title="Stories Title">
```

### 编写MDX文档

注：MDX在 IE11中编译会存在问题

Docs Block虽然将整个Docs页面拆分开来，进行自定义，但仍有一定的局限性，例如想要以一个story + 一段描述这样的形式来写文档，这对于Docs Block来说较难实现，此时，我们就可以考虑采用MDX语法来文档。MDX是一种结合了Markdown与JSX的标准文档格式，这使得我们可以在docs文档中自由编写标题与正文，并在其中穿插stories。

我们可以将现有的js文档修改为MDX文档，即将xxx.stories.js替换成xxx.stories.mdx文档，以达到灵活编写文档的目的。

从一个简单的例子来理解MDX代码：

```javascript
<!--- Badge.stories.mdx -->
import { Meta, Story, Canvas, argTypes, Props, ArgsTable } from '@storybook/addon-docs';

import { TeaBadge as Badge } from '../../web/components/Badge';

<Meta 
  title={"展示/徽章(Badge)"} 
  component={Badge}
/>

export const Template = (args) => <Badge {...args} />

# Badge

我们可以直接定义一个story，如下所示。

<Story name="success">
  <Badge theme="success" text="Success"/>
</Story>

同样，我们也可以在story外包裹一个`Canvas`

<Canvas>
  <Story name="warning">
      {Template.bind({})}
  </Story>
</Canvas>


<Props story="warning" />


我们还可以在一个区块里定义一堆story. 这些story会在文档的同一个区块里渲染显示，但每个单独的story都会生成一个标签.

<Canvas>
  <Story name="danger">
    <Badge theme="danger" text="Danger"/>
  </Story>
  <Story name="default">
    <Badge theme="default" text="Default"/>
  </Story>
  <Story name="dot-type">
    <Badge theme="default" pattern="dot"/>
  </Story>
  <Story name="ring-type">
    <Badge theme="default" pattern="ring"/>
  </Story>
</Canvas>
```

生成的文档效果如下图

![采用mdx编写的文档](https://github.com/Dasiylight/note/blob/master/picture/mdx.png?raw=true)

MDX与js对组件的引入方式类似，只是将js中`export default`替换为MDX的`<Meta>`标签，需要注意的是，MDX中的标签都需要提前引入。我们同样可以暴露一个基本的组件Template,并且调用它作为一个story。不同的是，MDX可以更灵活的对story进行展示：可以单独定义一个story展示，也可以将story放入`Canvas`中展示，可以展示组件的代码，并且可以通过argsTable更改组件。也可以将多个组件放入同一个`Canvas`中一起展示...

总之，MDX的编写方式与文档更加相似，因此如果需要一份详细的组件文档描述，采用MDX格式来编写或许是更好的选择。