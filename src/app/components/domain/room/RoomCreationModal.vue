<script lang="ts" setup>
import { ref, reactive } from "vue";
import { useRouter } from "vue-router";
import { useFormModal } from "@/app/components/ui/modal";
import type { FormInstance, FormRules } from "element-plus";
import { useProvider } from "@/app/platform";
import { RoomAPI } from "@/modules/room/services/RoomAPI";
import { RoomService } from "@/modules/room/services/RoomService";
import { ElMessage } from "element-plus";

const [roomApi, roomService] = useProvider([RoomAPI, RoomService]);
const form = ref<FormInstance | null>(null);
const loading = ref(false);
const router = useRouter();

const formRules = reactive<FormRules>({
  name: [
    {
      required: true,
      message: "Veuillez spécifier le nom du salon.",
      trigger: "blur"
    }
  ],
});

const { isVisible, hide, show, formModel } = useFormModal(
  {
    name: ""
  },
  form
);


async function onSubmit(form?: FormInstance) {
  if (!form) {
    return;
  }

  try {
    roomApi.leave("b871101-ac99-4712-a5b6-60d1216a44fe");
    loading.value = true;
    await form.validate();
    if(await roomApi.exists(formModel.value.name)) {
      ElMessage({message: "Le salon existe déjà", type: "error"});
      throw new Error("RoomAlreadyExistsError");
    }
    roomService.create({name: formModel.value.name});
    hide();
  } catch (e) {
    return;
  } finally {
    loading.value = false;
  }
}

defineExpose({
  show,
  hide
});
</script>

<template>
  <el-dialog v-model="isVisible" title="Création d'un nouveau salon" width="30%">
    <el-form
      ref="form"
      :model="formModel"
      :rules="formRules"
      label-position="top"
      class="login-form"
      @submit.prevent="onSubmit(form!)"
    >
      <el-form-item label="Nom du salon" prop="name">
        <el-input v-model="formModel.name"/>
      </el-form-item>
    </el-form>

    <template #footer>
      <div class="form-actions">
        <el-button native-type="reset" @click="hide()">Annuler</el-button>
        
        <el-button type="primary" native-type="submit" :loading="loading" @click="onSubmit(form!)">
          Créer le salon
        </el-button>
      </div>
    </template>
  </el-dialog>
</template>
