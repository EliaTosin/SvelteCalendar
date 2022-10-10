<script>
    export function createModalAndShow(event) {
		let modal = document.getElementById("eventModal");
		let span = document.getElementsByClassName("close")[0];

		modal.style.display = "block";
		span.onclick = function() {
			modal.style.display = "none";
		}

		window.onclick = function(event) {
			if (event.target == modal) {
				modal.style.display = "none";
			}
		}

		document.getElementById("modalText").innerHTML =
		"Title: " + event.title + "<br>" +
		"Description: " + event.description + "<br>" +
		"Date: " + event.date + "<br>" +
		"Duration: " + event.timeStart + " - " + event.timeEnd + "<br>" +
		"Calendar: " + event.calendar
	}
</script>

<div id="eventModal" class="modal">
    <div class="modal-content">
        <span class="close">&times;</span>
        <p id="modalText">Some text</p>
    </div>
</div>

<style>
    	.modal {
		display: none; 
		position: fixed; 
		z-index: 1; 
		padding-top: 100px; 
		left: 0;
		top: 0;
		width: 100%; 
		height: 100%;
		overflow: auto;
		background-color: rgb(0, 0, 0);
		background-color: rgba(0, 0, 0, 0.4);
	}

	.modal-content {
		background-color: black;
		color: white;
		margin: auto;
		padding: 20px;
		border: 1px solid orangered;
		width: 80%;
	}

	.close {
		color: #aaaaaa;
		float: right;
		font-size: 28px;
		font-weight: bold;
	}

	.close:hover,
	.close:focus {
		color: #000;
		text-decoration: none;
		cursor: pointer;
	}
</style>