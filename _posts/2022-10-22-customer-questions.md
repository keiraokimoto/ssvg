---
title: Customer Questions
layout: base
description: Our Frontend talking to Backend Python application serving questions.  This api allows us to get customer responses. 
permalink: /data/customer
image: /images/feedback.jpeg
tags: [javascript, fetch, dom, getElementID, appendChild]
---

{% include nav_data.html %}

<!-- HTML table fragment for page -->
<table>
  <thead>
  <tr>
    <th>Question</th>
    <th>Yes</th>
    <th>No</th>
  </tr>
  </thead>
  <tbody id="result">
    <!-- javascript generated data -->
  </tbody>
</table>

<!-- Script is layed out in a sequence (without a function) and will execute when page is loaded -->
<script>

  // prepare HTML defined "result" container for new output
  const resultContainer = document.getElementById("result");

  // keys for joke reactions
  const YES = "yes";
  const NO = "no";

  // prepare fetch urls
  var url = "http://localhost:5000/api/quiz/customers";

  const like_url = url + "/like/";  // yes reaction
  const no_url = url + "/no/";  // no reaction

  // prepare fetch GET options
  const options = {
    method: 'GET',  
    mode: 'cors',  
    cache: 'default', 
    credentials: 'omit',  
    headers: {
      'Content-Type': 'application/json'
    },
  };
  
  // prepare fetch PUT options, clones with JS Spread Operator (...)
  const put_options = {...options, method: 'PUT'}; // clones and replaces method

  // fetch the API
  fetch(url, options)
    // response is a RESTful "promise" on any successful fetch
    .then(response => {
      // check for response errors
      if (response.status !== 200) {
          error('GET API response failure: ' + response.status);
          return;
      }
      // valid response will have JSON data
      response.json().then(data => {
          console.log(data);
          for (const row of data) {
            // make "tr element" for each "row of data"
            const tr = document.createElement("tr");
            
            // td for question cell
            const question = document.createElement("td");
              question.innerHTML = row.id + ". " + row.question;  // add fetched data to innerHTML

            // td for yes cell with onclick actions
            const yes = document.createElement("td");
              const yes_but = document.createElement('button');
              yes_but.id = YES+row.id   // establishes a YEES JS id for cell
              yes_but.innerHTML = row.yes;  // add fetched "yes count" to innerHTML
              yes_but.onclick = function () {
                // onclick function call with "like parameters"
                reaction(YES, like_url+row.id, yes_but.id);  
              };
              yes.appendChild(yes_but);  // add "yes button" to yes cell

            // td for NO cell with onclick actions
            const no = document.createElement("td");
              const no_but = document.createElement('button');
              no_but.id = NO+row.id  // establishes a NO JS id for cell
              no_but.innerHTML = row.no;  // add fetched "no count" to innerHTML
              no_but.onclick = function () {
                // onclick function call with "worst parameters"
                reaction(NO, no_url+row.id, no_but.id);  
              };
              no.appendChild(no_but);  // add "boohoo button" to boohoo cell
             
            // this builds ALL td's (cells) into tr (row) element
            tr.appendChild(question);
            tr.appendChild(yes);
            tr.appendChild(no);

            // this adds all the tr (row) work above to the HTML "result" container
            resultContainer.appendChild(tr);
          }
      })
  })
  // catch fetch errors (ie Nginx ACCESS to server blocked)
  .catch(err => {
    error(err + " " + url);
  });

  // Reaction function to likes or jeers user actions
  function reaction(type, put_url, elemID) {

    // fetch the API
    fetch(put_url, put_options)
    // response is a RESTful "promise" on any successful fetch
    .then(response => {
      // check for response errors
      if (response.status !== 200) {
          error("PUT API response failure: " + response.status)
          return;  // api failure
      }
      // valid response will have JSON data
      response.json().then(data => {
          console.log(data);
          // Likes or Jeers updated/incremented
          if (type === YES) // like data element
            document.getElementById(elemID).innerHTML = data.yes;  // fetched yes data assigned to haha Document Object Model (DOM)
          else if (type === NO) // jeer data element
            document.getElementById(elemID).innerHTML = data.no;  // fetched boohoo data assigned to no Document Object Model (DOM)
          else
            error("unknown type: " + type);  // should never occur
      })
    })
    // catch fetch errors (ie Nginx ACCESS to server blocked)
    .catch(err => {
      error(err + " " + put_url);
    });
    
  }

  // Something went wrong with actions or responses
  function error(err) {
    // log as Error in console
    console.error(err);
    // append error to resultContainer
    const tr = document.createElement("tr");
    const td = document.createElement("td");
    td.innerHTML = err;
    tr.appendChild(td);
    resultContainer.appendChild(tr);
  }

</script>
