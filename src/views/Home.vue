<template>
  <div class="home">
    // Types const
    <pre>{{ codeType }}</pre>
    // Types interface
    <pre>{{ codeTypeInterface }}</pre>
    // Types class
    <pre>{{ codeTypeInterfaceResult }}</pre>
    // Actions
    <pre>{{ codeAction }}</pre>
    // Types state type
    <pre>{{ codeStateType }}</pre>
    // Init state
    <pre>{{ codeInitState }}</pre>
    // Code state
    <pre>{{ codeState }}</pre>
    // Selectors
    <pre>{{ codeSelector }}</pre>
    // Use selectors
    <pre>{{ codeSelectorUse }}</pre>
  </div>
</template>

<script lang="ts">
import _ from "lodash";
import { Options, Vue } from "vue-class-component";

export interface Redux {
  name: string;
  events: ReduxEvent[];
}

export interface ReduxType {
  name: string;
  type: string;
  nullable: boolean;
  default: string;
  condition?: string;
}

export interface ReduxEvent {
  name: string;
  type: string;
  payload: ReduxType[];
}

@Options({
  components: {},
})
export default class Home extends Vue {
  redux: Redux = {
    name: "RequestLive",
    events: [
      {
        name: "Mvz",
        type: "update",
        payload: [
          {
            name: "mvz",
            type: "string",
            default: '""',
            nullable: false,
          },
          {
            name: "approver3",
            type: "UserShort",
            nullable: true,
            default: "undefined",
          },
        ],
      },
      {
        name: "trip_type",
        type: "update",
        payload: [
          {
            name: "trip_type",
            type: "RequestTripType",
            nullable: false,
            default: "RequestTripType.BUSINESSTRIP",
          },
        ],
      },
      {
        name: "trip_direction",
        type: "update",
        payload: [
          {
            name: "trip_direction",
            type: "RequestTripDirection",
            nullable: false,
            default: "RequestTripDirection.ANY",
          },
          {
            name: "approver2",
            type: "UserShort",
            nullable: true,
            default: "undefined",
            condition: "trip_direction == RequestTripDirection.ABROAD",
          },
        ],
      },
      {
        name: "is_rhr",
        type: "update",
        payload: [
          {
            name: "is_rhr",
            type: "boolean",
            nullable: false,
            default: "false",
          },
        ],
      },
      {
        name: "seconded",
        type: "update",
        payload: [
          {
            name: "seconded",
            type: "User",
            nullable: true,
            default: "undefined",
          },
        ],
      },
    ],
  };

  getActionNameConst(event: ReduxEvent, splitByName: boolean = false) {
    if (splitByName)
      return (
        _.upperCase(this.getNameRedux(this.redux, "")).split(" ").join("_") +
        "/" +
        _.upperCase(`${event.type}_${event.name}`).split(" ").join("_")
      );

    return _.upperCase(
      `${this.getNameRedux(this.redux, "")}_${event.type}_${event.name}`
    )
      .split(" ")
      .join("_");
  }

  getActionInterfaceName(event: ReduxEvent) {
    return _.startCase(_.camelCase(this.getActionNameConst(event) + "Action"))
      .split(" ")
      .join("");
  }

  getActionName(event: ReduxEvent) {
    return _.camelCase(_.camelCase(this.getActionNameConst(event) + "Action"))
      .split(" ")
      .join("");
  }

  getNameVariableType(item: ReduxType) {
    return `${item.name}${item.nullable ? "?" : ""}: ${item.type}`;
  }

  getNameRedux(item: Redux, postfix: string = "State") {
    return _.startCase(item.name + postfix)
      .split(" ")
      .join("");
  }

  getReducerName(item: Redux) {
    return _.camelCase(_.camelCase(item.name)).split(" ").join("");
  }

  getNameStateInterface(item: Redux) {
    return `I${this.getNameRedux(item, "State")}`;
  }

  eventReset(): ReduxEvent {
    return {
      name: "reset",
      type: "update",
      payload: _.flattenDeep(this.redux.events.map((item) => item.payload)),
    };
  }

  get allEvents(): ReduxEvent[] {
    if (this.redux.events) return [...this.redux.events, this.eventReset()];

    return [];
  }

  get codeType() {
    return `${this.allEvents
      .map(
        (item) =>
          `const ${this.getActionNameConst(item)} = "${this.getActionNameConst(
            item,
            true
          )}";`
      )
      .join("\n")}
`;
  }

  get codeTypeInterface() {
    return `${this.allEvents
      .map(
        (item) => `interface ${this.getActionInterfaceName(item)} {
  type: typeof ${this.getActionNameConst(item)};
  ${item.payload
    .map((item) => `${this.getNameVariableType(item)};`)
    .join("\n  ")}
}`
      )
      .join("\n\n")}
`;
  }

  get codeTypeInterfaceResult() {
    return `export type ${this.getNameRedux(this.redux, "ActionType")} = 
  ${this.allEvents
    .map((item) => `| ${this.getActionInterfaceName(item)}`)
    .join("\n  ")}`;
  }

  get codeStateType() {
    return `export interface I${this.getNameRedux(this.redux)} {
  ${_.flattenDeep(this.redux.events.map((item) => item.payload))
    .map((item) => this.getNameVariableType(item))
    .join("\n  ")}
}`;
  }

  get codeAction() {
    return this.allEvents
      .map(
        (item) => `export function ${this.getActionName(
          item
        )}(params: {${item.payload
          .map((item) => this.getNameVariableType(item))
          .join(", ")}}): ${this.getActionInterfaceName(item)} {
  return {
    type: ${this.getActionNameConst(item)},
    ...params,
  };
}`
      )
      .join("\n\n");
  }

  get codeInitState() {
    return `const initialState: ${this.getNameStateInterface(this.redux)} = {
  ${_.flattenDeep(this.redux.events.map((item) => item.payload))
    .map((item) => `${item.name}: ${item.default},`)
    .join("\n  ")}
}`;
  }

  get codeState() {
    return `export function ${this.getNameRedux(
      this.redux
    )}Reducer(state = initialState, action: ${this.getNameRedux(
      this.redux,
      "ActionType"
    )}): ${this.getNameStateInterface(this.redux)} {
  switch (action.type) {
    ${this.allEvents
      .map(
        (item) => `case ${this.getActionNameConst(item)}:
      return {
        ...state,
        ${item.payload
          .map((item) => `${item.name}: action.${item.name}`)
          .join(",\n        ")}
      }
`
      )
      .join("\n    ")}
    default:
      return state;
  }
}
`;
  }

  get codeSelector() {
    return _.flattenDeep(this.redux.events.map((item) => item.payload))
      .map(
        (item) =>
          `export const ${this.getUseSelectorName(
            this.redux,
            item
          )} = (state: RootState) => state.${this.getReducerName(this.redux)}.${
            item.name
          };\n`
      )
      .join("");

    return `export const selectRequestLiveMvz = (state: RootState) => state.requestLive.mvz;`;
  }

  getUseSelectorName(redux: Redux, event: ReduxType) {
    return `select${this.getNameRedux(redux, "")}${_.startCase(event.name)
      .split(" ")
      .join("")}`;
  }

  getSelectorName(redux: Redux, event: ReduxType) {
    const nemeRedux = _.camelCase(_.camelCase(redux.name)
      .split(" ")
      .join(""));

      const nameEvent = _.startCase(event.name)
      .split(" ")
      .join("");

    return `${nemeRedux}${nameEvent}`;
  }

  get codeSelectorUse() {
    return '/*\n' + _.flattenDeep(this.redux.events.map((item) => item.payload))
      .map((item) => `const ${this.getSelectorName(this.redux, item)} = useSelector(${this.getUseSelectorName(this.redux, item)});\n`)
      .join("") + '*/';

    return `const requestLiveIsRhr = useSelector(selectRequestLiveIsRhr);`;
  }
}
</script>
