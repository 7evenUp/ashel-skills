# ashel-skills

Author's code conventions for AI coding agents.

## Ashel Front Style

[Ashel Front Style](skills/ashel-front-style/SKILL.md) applies Ashel's React and TypeScript code conventions: semantic import and state groups, compact components, explicit handlers, Tailwind, and typed APIs. It guides code organization and implementation choices; the product brief determines the visual design.

Use it when you want Ashel's conventions or your project already adopts them. Existing project instructions, formatting, architecture, and dependencies take priority unless you explicitly request a change. Library choices in examples are optional.

The skill instructions are written in Russian, with English code identifiers.

## Install

Once the repository is public on GitHub, install with Node.js/npm:

```sh
npx skills add 7evenUp/ashel-skills --skill ashel-front-style
```

Install globally for Codex:

```sh
npx skills add 7evenUp/ashel-skills --skill ashel-front-style --agent codex --global
```

## Use

```text
Use $ashel-front-style to implement a profile page in this React/TypeScript project.
Preserve the existing formatter, component library, and API contracts.
```

For a new project:

```text
Use $ashel-front-style to build a React landing page with Tailwind.
Compose it from sections and keep static content local to each section.
```

## Example: group state by purpose

Before, the step flag interrupts the form fields:

```tsx
const [oldEmailInput, setOldEmailInput] = useState("")
const [isSecondStep, setIsSecondStep] = useState(false)
const [newEmailInput, setNewEmailInput] = useState("")
```

After, navigation state and form fields form separate groups:

```tsx
const [isSecondStep, setIsSecondStep] = useState(false)

const [oldEmailInput, setOldEmailInput] = useState("")
const [newEmailInput, setNewEmailInput] = useState("")
```

Both fragments belong inside a React component with `useState` imported from React. Independent values remain separate states; grouping does not require an object or reducer.

## Contents

- [SKILL.md](skills/ashel-front-style/SKILL.md): conventions and project precedence.
- [Component organization](skills/ashel-front-style/references/component-organization.md): state, handlers, resets, and query groups.
- [Code examples](skills/ashel-front-style/references/examples.md): `cn`, Skeleton, and a typed RTK Query API.
- [Codex metadata](skills/ashel-front-style/agents/openai.yaml): display name and description.

## skills.sh

According to the [skills.sh FAQ](https://skills.sh/docs/faq), leaderboard entries are tracked through CLI installations from public GitHub repositories with telemetry enabled.

## License

[MIT](LICENSE).
