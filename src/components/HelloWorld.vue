<template>
  <h1>{{ msg }}</h1>
  <button @click="count++">count is: {{ count }}</button>
  <div>{{ fcMembers }}</div>
  <p>
    Edit <code>components/HelloWorld.vue</code> to test hot module replacement.
  </p>
</template>

<script>
import axios, * as others from "axios";
const fcName = "Kyng's Krew";
const serverName = "Midgardsormr";
const mountName = "Round Lanner";

async function getChars(fcMembers) {
  let mimo = {};
  let i = 1;
  try {
    for (let fcMember of fcMembers) {
      // mimo = [...mimo, await xiv.character.get(element.ID, { data: 'MIMO' })];
      const charQueryStr =
        "https://xivapi.com/character/" + fcMember.ID + "?data=MIMO";
      const char = await axios.get(charQueryStr);
      console.log(
        `downloading ${fcMember.Name} | total progress: ${i}/${fcMembers.length}`
      );
      mimo[char.data.Character.Name] = char.data.Mounts;
      i += 1;
    }
    return mimo;
  } catch (error) {
    console.error(error);
  }
}

async function filterByMount() {
  try {
    const fcQueryStr =
      "https://xivapi.com/freecompany/search?name=" +
      fcName +
      "&server=" +
      serverName;
    const response = await axios.get(fcQueryStr);
    //const response = await xiv.freecompany.search(fcName, { server: serverName });
    const fcID = response.data.Results[0].ID;
    const fcMemberQuery =
      "https://xivapi.com/freecompany/" + fcID + "?data=FCM";
    const fcResponse = await axios.get(fcMemberQuery);
    const fcMembers = fcResponse.data.FreeCompanyMembers;
    console.log(fcMembers);
    const mimo = await getChars(fcMembers);
    console.log(mimo);
    const fcMemberNames = [];
    for (let fcMember of fcMembers) {
      fcMemberNames.push(fcMember.Name);
    }

    const filter = fcMemberNames.filter((fcMember) => {
      for (let mount of mimo[fcMember]) {
        if (mountName === mount.Name) {
          return true;
        }
        continue;
      }
    });
    console.log(filter);
    return filter;
  } catch (error) {
    console.error(error);
  }
}

(async function () {
  const fcMembers = await filterByMount();
  console.log(fcMembers);
})();
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },
  data() {
    return {
      count: 0,
    };
  },
};
</script>
