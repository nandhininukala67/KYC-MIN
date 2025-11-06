private void parseResponse(HttpResponseWrapper responseWrapper) throws Exception {
    if (responseWrapper == null) {
        requestSuccess = false;
        message = "Response is blank";
        return;
    }

    statusCode = responseWrapper.getHttpStatusCode();
    evt.addMessage("Status code - " + statusCode);

    if (statusCode != 200 || responseWrapper.getResponseBody() == null) {
        requestSuccess = false;
        message = "HTTP Error or blank response";
        return;
    }

    try {
        responsePayload = new JSONObject(responseWrapper.getResponseBody());
    } catch (Exception e) {
        requestSuccess = false;
        message = "Response not in JSON format: " + e.getMessage();
        evt.addMessage(ExceptionUtils.getStackTrace(e));
        return;
    }

    evt.addMessage("Response plain payload - " + responsePayload);

    if (responsePayload.has("ERROR_CODE")) {
        requestSuccess = false;
        cbsIrc = responsePayload.optString("ERROR_CODE");
        cbsIrcDetails = responsePayload.optString("ERROR_DESCRIPTION");
        evt.addMessage("Error: " + cbsIrc + " - " + cbsIrcDetails);
        return;
    }

    actualApiResponse = responsePayload.optString("RESPONSE_STATUS", "UNKNOWN");
    cbsIrc = actualApiResponse;

    if (!StringUtils.equals(actualApiResponse, cfg.get("success-response-status", "0"))) {
        requestSuccess = false;
        message = "Transaction failed";
        return;
    }

    parseAPIResponse();
}
