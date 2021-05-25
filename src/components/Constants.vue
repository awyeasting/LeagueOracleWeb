<script>
var production= false
export default {
	HOSTNAME: (production ? "http://sermire.com:5000" : "http://localhost:5000")
}
</script>