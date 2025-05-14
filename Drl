package com.rbs.bdd.application.service;

import com.rbs.bdd.generated.ValidateArrangementForPaymentRequest;
import jakarta.xml.bind.JAXBContext;
import jakarta.xml.bind.Unmarshaller;
import org.junit.jupiter.api.Test;
import org.springframework.ws.WebServiceMessage;
import org.springframework.ws.soap.saaj.SaajSoapMessage;
import org.springframework.ws.test.support.MockWebServiceMessage;
import org.w3c.dom.Document;
import org.w3c.dom.Node;

import javax.xml.parsers.DocumentBuilderFactory;
import javax.xml.transform.TransformerFactory;
import javax.xml.transform.stream.StreamResult;
import javax.xml.xpath.XPath;
import javax.xml.xpath.XPathConstants;
import javax.xml.xpath.XPathFactory;
import java.io.ByteArrayOutputStream;
import java.io.InputStream;

import static org.junit.jupiter.api.Assertions.*;

public class AccountValidationServiceTest {

    private final AccountValidationService accountValidationService = new AccountValidationService();

    @Test
    public void testValidateSchemaAndBusinessLogic_modifiesSoapResponseCorrectly() throws Exception {
        // Load static SOAP request from test resources
        InputStream requestXml = getClass().getClassLoader().getResourceAsStream("static-request/static-request.xml");
        assertNotNull(requestXml, "static-request.xml not found");

        // Unmarshal request to Java object
        JAXBContext context = JAXBContext.newInstance(ValidateArrangementForPaymentRequest.class);
        Unmarshaller unmarshaller = context.createUnmarshaller();
        ValidateArrangementForPaymentRequest request = (ValidateArrangementForPaymentRequest) unmarshaller.unmarshal(requestXml);

        // Prepare a mock WebServiceMessage for response
        ByteArrayOutputStream responseOutputStream = new ByteArrayOutputStream();
        WebServiceMessage message = new MockWebServiceMessage(responseOutputStream);
        ((SaajSoapMessage) message).setSoapAction("testAction"); // optional, avoids NullPointerException in some edge cases

        // Act: Validate schema and apply business logic
        accountValidationService.validateSchema(request);
        accountValidationService.validateBusinessRules(request, message);

        // Transform the written SOAP response into a Document
        Document responseDoc = DocumentBuilderFactory.newInstance()
                .newDocumentBuilder()
                .parse(new java.io.ByteArrayInputStream(responseOutputStream.toByteArray()));

        XPath xpath = XPathFactory.newInstance().newXPath();

        // Assert that business logic was applied (systemId and transactionId modified)
        Node systemId = (Node) xpath.evaluate("//*[local-name()='responseId']/*[local-name()='systemId']", responseDoc, XPathConstants.NODE);
        assertNotNull(systemId);
        assertEquals("ModifiedESP", systemId.getTextContent());

        Node transactionId = (Node) xpath.evaluate("//*[local-name()='responseId']/*[local-name()='transactionId']", responseDoc, XPathConstants.NODE);
        assertNotNull(transactionId);
        assertEquals("ModifiedTxn123", transactionId.getTextContent());
    }
}
