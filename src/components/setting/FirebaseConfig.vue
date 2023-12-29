<script lang="js">
// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
import { getFirestore } from "firebase/firestore";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyCthbu3VjunHQ-0aW2bHIVHuoZTdNTfSb8",
  authDomain: "dbms-final-ecf70.firebaseapp.com",
  projectId: "dbms-final-ecf70",
  storageBucket: "dbms-final-ecf70.appspot.com",
  messagingSenderId: "305919680900",
  appId: "1:305919680900:web:2e4ed34e1b2d4243813dd1",
  measurementId: "G-T8GEE95HR7"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
const db = getFirestore(app);
export default app;
</script>
