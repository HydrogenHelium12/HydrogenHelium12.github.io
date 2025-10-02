<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Select School & Student</title>
</head>
<body>
  <h1>Choose School and Student</h1>

  <label for="schoolSelect">Select School:</label>
  <select id="schoolSelect">
    <option value="">-- choose a school --</option>
  </select>

  <br><br>

  <label for="studentSelect">Select Student:</label>
  <select id="studentSelect">
    <option value="">-- choose a student --</option>
  </select>

  <br><br>

  <button id="submitBtn">Submit</button>
  <p id="statusMsg" style="color: green;"></p>

  <script>
    const endpoint = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec'; // Replace with your Web App URL

    let data = null;

    async function fetchData() {
      const resp = await fetch(endpoint);
      if (!resp.ok) {
        console.error('Failed to fetch data', resp.status, resp.statusText);
        return;
      }
      data = await resp.json();
      populateSchools();
    }

    function populateSchools() {
      const schoolSelect = document.getElementById('schoolSelect');
      schoolSelect.innerHTML = '<option value="">-- choose a school --</option>';
      data.schools.forEach(schoolName => {
        const opt = document.createElement('option');
        opt.value = schoolName;
        opt.text = schoolName;
        schoolSelect.appendChild(opt);
      });
    }

    function onSchoolChange() {
      const schoolSelect = document.getElementById('schoolSelect');
      const studentSelect = document.getElementById('studentSelect');
      const selectedSchool = schoolSelect.value;

      studentSelect.innerHTML = '<option value="">-- choose a student --</option>';

      if (selectedSchool && data.studentsBySchool[selectedSchool]) {
        data.studentsBySchool[selectedSchool].forEach(studentName => {
          const opt = document.createElement('option');
          opt.value = studentName;
          opt.text = studentName;
          studentSelect.appendChild(opt);
        });
      }
    }

    async function submitForm() {
      const school = document.getElementById('schoolSelect').value;
      const student = document.getElementById('studentSelect').value;
      const statusMsg = document.getElementById('statusMsg');

      if (!school || !student) {
        statusMsg.style.color = 'red';
        statusMsg.textContent = 'Please select both school and student.';
        return;
      }

      try {
        const resp = await fetch(endpoint, {
          method: 'POST',
          body: JSON.stringify({ school, student }),
          headers: {
            'Content-Type': 'application/json'
          }
        });

        const result = await resp.json();

        if (result.status === 'success') {
          statusMsg.style.color = 'green';
          statusMsg.textContent = 'Submission successful!';
        } else {
          throw new Error(result.message || 'Unknown error');
        }
      } catch (error) {
        console.error('Submission error:', error);
        statusMsg.style.color = 'red';
        statusMsg.textContent = 'Submission failed: ' + error.message;
      }
    }

    document.getElementById('schoolSelect').addEventListener('change', onSchoolChange);
    document.getElementById('submitBtn').addEventListener('click', submitForm);

    fetchData().catch(err => console.error('Error fetching data:', err));
  </script>
</body>
</html>
